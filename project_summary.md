# Project Title
Automated Digital Photo Collage

## Authors
-Tommy Mintz https://github.com/TommyMintz/

## Description
An interactive collage using a raspberry pi and a webcam 

## Link to Prototype
NOTE: If your project lives online you can add one or more links here. Make sure you have a stable version of your project running before linking it.

[Documentation](http://tommymintz.com/adpc/ "Documentation")

## Example Code
NOTE: Wrap your code blocks or any code citation by using ``` like the example below.
```
#!/usr/bin/python
#
#The Automated Digital Photo Collage
#A program that renders a time lapse collage
#
#http://tommymintz.com/adpc/
#
#Released under the GNU General Public Liscence
#
#January 2013
#
#ver 1.1.25

import pygame
import pygame.camera
import numpy
from pygame.locals import *
import sys
import os
import subprocess
import time

#creates 'numgen.py' and sets numgen.imgno=1 if numgen.py file is not found.
sys.path.append('~/')
try:
  import numgen
  print 'numgen file found. numgen.imgno =', numgen.imgno
except ImportError, err:
  print 'no numgen file found. generated new sequence.'
  f = open('numgen.py', 'w')
  initialwrite = f.write('imgno = 1\n')
  f.close()
  import numgen
  print 'numgen imagenumber is now', numgen.imgno  




# adds 1 to the numgen.imgno variable and writes it to the numgen.py file... set to 'a' on keyboard in main loop
def adder():
  f = open('numgen.py', 'w')
  numgen.imgno +=1
  initialwrite = f.write('imgno = %s' %(numgen.imgno))
  f.close()
  
  print 'numgen is now', numgen.imgno


#initialise camera
pygame.camera.init()
global cam
cam = pygame.camera.Camera('/dev/video0', (800,600))


#function for taking images from webcam, returns pygame surface
def shootpic():
  cam.start()
  img = cam.get_image()
  cam.stop()
  return img

#algorithm for blurring surfaces  
def blurSurf(surface, amt):
      if amt < 1.0:
        raise ValueError ("Arg 'amt' must be greater than 1.0")
      scale = 1.0/float(amt)
      surf_size = surface.get_size()
      scale_size = (int(surf_size[0]*scale), int(surf_size[1]*scale))
      surf = pygame.transform.smoothscale(surface, scale_size)
      surf = pygame.transform.smoothscale(surf, surf_size)
      return surf

#Toggle fullscreen with spacebar
def toggle_fullscreen():
  screen = pygame.display.get_surface()
  tmp = screen.convert()
  caption = pygame.display.get_caption()
  #cursor = pygame.mouse.get_cursor()

  w,h = screen.get_width(), screen.get_height()
  flags = screen.get_flags()
  bits = screen.get_bitsize()

  pygame.display.quit()
  pygame.display.init()

  screen = pygame.display.set_mode((w,h),flags^FULLSCREEN,bits)
    
  screen.blit(tmp,(0,0))
  pygame.display.set_caption(*caption)

  pygame.key.set_mods(0)
  pygame.mouse.set_visible(0)

  return screen


#----PRINT----# 
def printer():
  #printer() takes last imagecollage.jpg and prints  via lp?
  num = numgen.imgno - 3 
  print 'got num', num, 'numgen.imgno is', numgen.imgno
  try:
    printfeed = 'lp ~/imgcollage%s.jpg' % num
    print printfeed
    printer = subprocess.Popen([printfeed],shell=True)
    print 'print command sent'
  except Exception, e:
    print 'exception caught:', e, sys.exc_info()[0]

    
#-------COLLAGE--------#

#collage images from webcam
#takes img1 image from motion detect, returns 36th collage for printing as a pygame surface
def collage():
  shotcount = 0
  base = shootpic() #base made 
  print 'base made from img1'
  imgold = base
  imgold3d = pygame.surfarray.pixels3d(imgold)


  #------------Collage Loop ----------#
  
  while shotcount < 36:
      try:
          for event in pygame.event.get():
            if event.type == QUIT or (event.type == KEYDOWN and event.key == K_ESCAPE):
              sys.exit()
            if (event.type == KEYDOWN and event.key == K_SPACE):
              toggle_fullscreen()
              s = "Ready. fyi: numgen. imgno is now " + str(numgen.imgno)
              text = font.render(s, 1, (10, 10, 10))
              global background
              background = pygame.Surface(screen.get_size())
              background.blit(text, textpos)  # blit text to surface
              screen.blit(background, (0,0))  # blit surface to screen
              pygame.display.flip()
            if (event.type == KEYDOWN and event.key == K_p):
              printer()
            if (event.type == KEYDOWN and event.key == K_q):
              sys.exit()
          img = shootpic()  
          blurimg = blurSurf(img, 6)
          blurbase = blurSurf (base, 6)
          blurbasearray = pygame.PixelArray(blurbase)
          blurimgarray = pygame.PixelArray(blurimg)
          alphachannelarray = blurbasearray.compare(blurimgarray, distance=0.1, weights=(0.1, 0.1, 0.1))
          alphachannel = alphachannelarray.make_surface()
          bluralpha = blurSurf (alphachannel, 5)
          alphachannel3d = pygame.surfarray.pixels3d(bluralpha)
          img3d = pygame.surfarray.pixels3d(img)
          numpy.putmask(img3d, alphachannel3d == [0,0,0], imgold3d)             
          imgold3d = img3d
          imgcollage = pygame.surfarray.make_surface(img3d)
          imgcollage = imgcollage.convert()
          screen.blit(imgcollage, (0,0))  
          pygame.display.flip() 
          pygame.image.save (imgcollage,'imgcollage%d.jpg' %numgen.imgno )
          print ('imgcollage%d.jpg saved' %numgen.imgno )
          pygame.image.save (alphachannel,'alphachannel%d.jpg' %numgen.imgno )
          print ('alphachannel%d.jpg saved' %numgen.imgno )
          adder()
          shotcount +=1
          print 'shotcount is ', shotcount

          time.sleep(2)

                
      except Exception, e:
            print 'program would restart machine from collage loop', e
  
  return imgcollage

#-------MAIN--------#  


def main():
  #initialise scren
  os.environ['SDL_VIDEO_WINDOW_POS'] = "0,0"
  pygame.init()  
  global screen
  screen = pygame.display.set_mode ((800,600), pygame.FULLSCREEN)#opens pygame fullscreen
  #screen = pygame.display.set_mode ((800,600))#open pygame screen that is 800 x 600 pixels
  pygame.display.set_caption('ADPC')
  pygame.mouse.set_visible(0) #Hide mouse

  #fill backgroaund
  global background
  background = pygame.Surface(screen.get_size())
  background = background.convert()    # sets pixel format to match screen
  background.fill((255,255,255))  #make white rectangle size of background on background surface

  #display some text
  font = pygame.font.Font(None, 36)
  text = font.render("Hello. Just a moment, please.", 1, (10, 10, 10))
  textpos = text.get_rect()
  textpos.centerx = background.get_rect().centerx
  textpos.centery = background.get_rect().centery
  background.blit(text, textpos)  # blit text to surface

  screen.blit(background, (0,0))  # blit surface to screen

  pygame.display.flip()

  clock = pygame.time.Clock()


  background.fill((255,255,255))
  text = font.render("The Automated Digital Photo Collage is ready.", 1, (10, 10, 10))
  textpos = text.get_rect()
  textpos.centerx = background.get_rect().centerx
  textpos.centery = background.get_rect().centery
  background.blit(text, textpos)  # blit text to surface
  screen.blit(background, (0,0))  # blit surface to screen
  pygame.display.flip()
  screen.blit(background, (0,0))  # blit surface to screen
  pygame.display.flip()
  background.fill((255,255,255))
  clock.tick(1)
    
  temp = collage() #set temp to return of collage using img1 from motion detect...should be..36th collage
  screen.blit(temp, (0,0)) #should show 36th collage
  print 'print being sent now'
  printer()
  clock.tick(30000)#have program show last collage for a few seconds
            
  print 'autorestarting'
  executable = sys.executable
  args=sys.argv[:]
  args.insert(0, sys.executable)
  os.execvp(executable, args)
  
if __name__== '__main__': main()
```
## Links to External Libraries
 NOTE: You can also use this space to link to external libraries or Github repositories you used on your project.

[Example Link](http://www.google.com "Example Link")

## Images & Videos
NOTE: For additional images you can either use a relative link to an image on this repo or an absolute link to an externally hosted image.

![Two Views of ADPC](http://tommymintz.com/adpc/installation-and-street-view-of-adpc.jpg "ADPC views")

https://www.youtube.com/watch?v=30yGOxJJ2PQ
