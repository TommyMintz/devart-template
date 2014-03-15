# Project Title
Automated Digital Photo Collage

## Authors
Tommy Mintz https://github.com/TommyMintz/

## Description
An interactive collage using a raspberry pi and a webcam. Written in Python 

## Link to Prototype


[Documentation](http://tommymintz.com/adpc/ "Documentation")

## Sample Code

```




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


```


## Images & Videos


![Two Views of ADPC](http://tommymintz.com/adpc/installation-and-street-view-of-adpc.jpg "ADPC views")

[video of sequence of ADPC generated images](https://www.youtube.com/watch?v=vxeggjzgJmo “https://www.youtube.com/watch?v=vxeggjzgJmo”)
