# A guide for installing Softwares in Ubuntu

![](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQtbz9BVNuvNFXRXMnEz4K52oBPVhyAR6r-_t_5uQPyREZ3DDm7egje9-zXzkPf1d1wTg8&usqp=CAU)



**Table of Contents**

[TOCM]

[TOC]

# Media Player
1.  VLC player


	`sudo apt-get install vlc`

# Editor
1.  Geany


	`sudo apt-get install geany`

2. Vim


	`sudo apt-get install vim`

# Graphics Apps
1. Gimp


	`sudo apt-get install gimp`
	
# System Optimizer
1.  Stacer

	`sudo apt install stacer`
	
# Compiler
1. Fortran


	`sudo apt-get install gfortran`
	
# Miscellaneous
1. Tree


	`sudo apt-get install tree`
	
2. Git


	`sudo apt-get install git`
	
3. Tweak


	`sudo apt-get install gnome-tweaks`
	
4. Font Manager


	`sudo apt -y install font-manager`
	
5. Telegram


	`sudo snap install telegram-desktop`
	
6. TexLive


	`sudo apt install texlive-full`
	
7. VPN


	`sudo /sbin/modprobe tun`
	
	`sudo apt-get install openconnect`












import numpy as np
import pandas as pd

class Particle():
    def __init__(self, lowerBound,
                       upperBound,
                       objectiveFunction):
        
        super(Particle, self).__init__()
        
        self.lowerBound = lowerBound
        self.upperBound = upperBound
        self.objectiveFunction = objectiveFunction
        self.dimension = len(self.lowerBound)
        
        self.initialize()

        
    def initialize(self):
        
        '''
        Each particle has:
            1) position
            2) velocity (zero for first time)
            3) cost (function value)
            4) best position: position where best value of objective function was returned.
            5) best cost: best value of objective function which has been returned.
        '''
        
        self.position = []
        for i in range(self.dimension):
            if self.lowerBound[i] < self.upperBound[i]:
                self.position.append(np.random.randint(self.lowerBound[i], self.upperBound[i] + 1, 1, dtype=int))
            elif self.lowerBound[i]== self.upperBound[i]:
                self.position.append(np.array([self.lowerBound[i]], dtype=int))
                
            
        self.position = np.array(self.position)        
        self.velocity = np.zeros_like(self.position)
        self.cost = np.array(self.objectiveFunction(self.position))
        self.best_position = self.position
        self.best_cost = self.cost

    
    def updateVelocity(self, xBestGlobal, xBest, velocity, position):
        '''
        particle's velocity update.       

        Parameters
        ----------
        xBestGlobal : TYPE
            DESCRIPTION.
        xBest : TYPE
            DESCRIPTION.
        velocity : TYPE
            DESCRIPTION.
        position : TYPE
            DESCRIPTION.

        Returns
        -------
        velocity : TYPE
            DESCRIPTION.

        '''
        w = 0.9
        c1 = 2.0
        c2 = 2.0
        r1=np.random.uniform(*position.shape)
        r2=np.random.uniform(*position.shape)
        
        velocity = w*velocity + r1*c1*(xBest - position) + r2*c2*(xBestGlobal-position)
        
        velocity = velocity.astype(int)
        
        return velocity
        
    
    def updatePosition(self, velocity, position, best_cost, x_best_cost):
        
        new_position = position + velocity
        
        for d in range(self.dimension):
            if new_position[d] < self.lowerBound[d]:
                new_position[d] = self.lowerBound[d]
            elif new_position[d] > self.upperBound[d]:
                new_position[d] = self.upperBound[d]
        
        
        new_cost = self.objectiveFunction(new_position)
        
        if new_cost < best_cost:
            new_best_cost = new_cost
            new_best_position = new_position
        else:
            new_best_cost = best_cost
            new_best_position = x_best_cost                       
        
        return new_position, new_cost, new_best_cost, new_best_position

class PSO(object):
    def __init__(self, numberOfParticles = 10, maxIteration = 100):
        super(PSO, self).__init__()
        
        self.numberOfParticles = numberOfParticles
        self.maxIteration = maxIteration
        
    def run(self, function, lower_bound, upper_bound):
                       
        particles = self.initialize_particles(function, lower_bound, upper_bound)
        
        his = []
        his_f = []
        for ite in range(self.maxIteration):
            his_0 = []
            for p in range(self.numberOfParticles):
                if p == 0:
                    his_0 = particles[p].position
                else:
                    his_0 = np.concatenate([his_0, particles[p].position], axis=1)
                
                his_f.append(particles[p].cost)
            
            his.append(his_0)
            
            
            for p in range(self.numberOfParticles):
                
                particles[p].velocity = particles[p].updateVelocity(self.best_xGlobal, particles[p].best_position, particles[p].velocity, particles[p].position)
                particles[p].position, particles[p].cost, particles[p].best_cost, particles[p].best_poistion = particles[p].updatePosition(particles[p].velocity, particles[p].position, particles[p].best_cost, particles[p].best_position)
            
            for p in range(self.numberOfParticles):  
                if particles[p].best_cost < self.best_costGlobal:
                    self.best_costGlobal = particles[p].best_cost
                    self.best_xGlobal = particles[p].best_poistion
                        
                    
            print(ite, " cost=", self.best_costGlobal[0])        
        
        return self.best_xGlobal, self.best_costGlobal, his, his_f
  
    
    def initialize_particles(self, function, lower_bound, upper_bound):
                
        particles = []
        for p in range(self.numberOfParticles):
            particles.append(Particle(lower_bound, upper_bound, function))
        
        for p in range(self.numberOfParticles):
            
            if p == 0:
                self.best_costGlobal = particles[p].cost
                self.best_xGlobal = particles[p].position
                
            elif particles[p].cost < self.best_costGlobal:
                self.best_costGlobal = particles[p].cost
                self.best_xGlobal = particles[p].position
            
        
        return particles

        


def func(x0):
    x = x0[0]
    y = x0[1]
    z = x*y
    return z


p = PSO(4, 20)
x, f, his, hisf = p.run(func, [-20, -20], [21, 21])


# hisf2 = np.reshape(hisf, (10,4))

