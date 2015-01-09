package game;

import java.awt.Canvas;
import java.awt.Color;
import java.awt.Dimension;
import java.awt.Font;
import java.awt.Graphics2D;
import java.awt.Image;
import java.awt.event.MouseAdapter;
import java.awt.event.MouseEvent;
import java.awt.event.MouseMotionAdapter;
import java.awt.image.BufferStrategy;
import java.io.File;
import java.io.IOException;
import java.util.logging.Level;
import java.util.logging.Logger;
import javax.imageio.ImageIO;

import javax.swing.JFrame;
import javax.swing.JPanel;


public class Game implements Runnable{
   
   final int WIDTH = 800;
   final int HEIGHT = 540;
   
   JFrame frame;
   Canvas canvas;
   BufferStrategy bufferStrategy;
   
   public Game(){
      frame = new JFrame("River Crossing Game");
      
      JPanel panel = (JPanel) frame.getContentPane();
      panel.setPreferredSize(new Dimension(WIDTH, HEIGHT));
      panel.setLayout(null);
      
      canvas = new Canvas();
      canvas.setBounds(0, 0, WIDTH, HEIGHT);
      canvas.setIgnoreRepaint(true);
      
      panel.add(canvas);
      
      canvas.addMouseListener(new MouseControl());
      canvas.addMouseMotionListener(new MouseMotion());
      
      frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
      frame.pack();
      frame.setResizable(false);
      frame.setVisible(true);
      
      canvas.createBufferStrategy(2);
      bufferStrategy = canvas.getBufferStrategy();
      
      canvas.requestFocus();
   }
   
        
   private class MouseControl extends MouseAdapter{
      @Override
      public void mousePressed(MouseEvent e) {
          if(crossing == false){
              try {
                  if(cX < e.getX() && e.getX() < (cX + cW) && cY < e.getY() && e.getY() < (cY + cH)){
                      selected = 3;
                      if(isRight[selected] == isRight[4]){
                          charDragged = true;
                          x = cX;
                          y = cY;
                      } else {
                          selected = -1;
                      }
                  } else if(wX < e.getX() && e.getX() < (wX + wW) && wY < e.getY() && e.getY() < (wY + wH)){
                      selected = 1;
                      if(isRight[selected] == isRight[4]){
                          charDragged = true;
                          x = wX;
                          y = wY;
                      } else {
                          selected = -1;
                      }
                  } else if(sX < e.getX() && e.getX() < (sX + sW) && sY < e.getY() && e.getY() < (sY + sH)){
                      selected = 2;
                      if(isRight[selected] == isRight[4]){
                          charDragged = true;
                          x = sX;
                          y = sY;
                      } else {
                          selected = -1;
                      }
                  } else if(fX < e.getX() && e.getX() < (fX + fW) && fY < e.getY() && e.getY() < (fY + fH)) {
                      selected = 0;
                      if(isRight[selected] == isRight[4]){
                          charDragged = true;
                          x = fX;
                          y = fY;
                      } else {
                          selected = -1;
                      }
                  }
                  x1 = e.getX();
                  y1 = e.getY();
                  
                  File tempFile;
                  if(isRight[selected] == false){
                      if(selected == 0){
                          tempFile = new File("test\\img\\farmer\\carryL.png");
                      } else if(selected == 1){
                          tempFile = new File("test\\img\\wolf\\carryL.png");
                      } else if(selected == 2){
                          tempFile = new File("test\\img\\sheep\\carryL.png");
                      } else if(selected == 3){
                          tempFile = new File("test\\img\\cabbage\\carryL.png");
                      } else {
                          tempFile = new File("");
                      }
                  } else {
                      if(selected == 0){
                          tempFile = new File("test\\img\\farmer\\carryR.png");
                      } else if(selected == 1){
                          tempFile = new File("test\\img\\wolf\\carryR.png");
                      } else if(selected == 2){
                          tempFile = new File("test\\img\\sheep\\carryR.png");
                      } else if(selected == 3){
                          tempFile = new File("test\\img\\cabbage\\carryR.png");
                      } else {
                          tempFile = new File("");
                      }
                  }
                  carry = ImageIO.read(tempFile);
              } catch (IOException ex) {
                  Logger.getLogger(Game.class.getName()).log(Level.SEVERE, null, ex);
              }
          }
      }
      @Override
      public void mouseReleased(MouseEvent e) {
        if(charDragged == true){
            if(isInBoat[selected] == false && selected != 0){
                if(isInBoat[1] || (isInBoat[2] || isInBoat[3])){
                    outOfBoat = true;
                } else if(x1 > bX && x1 < bX + bW && y1 > bY - 50){
                    intoBoat = true;
                }
            } else {
                if(x1 > bX && x1 < bX + bW && y1 > bY - 50){
                    intoBoat = true;
                } else {
                    outOfBoat = true;
                }
            }
            
            charDragged = false;
        }
      }

      @Override
      public void mouseClicked(MouseEvent e) {
          crossing = true;
      }
    }
    class MouseMotion extends MouseMotionAdapter {
      @Override
      public void mouseDragged(MouseEvent e) {
        x2 = e.getX();
        y2 = e.getY();
        x = x + x2 - x1;
        y = y + y2 - y1;
        x1 = x2;
        y1 = y2;
        
      }

   }
   
   long desiredFPS = 60;
   long desiredDeltaLoop = (1000*1000*1000)/desiredFPS;
    
   boolean running = true;
   
   public void run(){
      
      long beginLoopTime;
      long endLoopTime;
      long currentUpdateTime = System.nanoTime();
      long lastUpdateTime;
      long deltaLoop;
      
      while(running){
         beginLoopTime = System.nanoTime();
         
         render();
         
         lastUpdateTime = currentUpdateTime;
         currentUpdateTime = System.nanoTime();
          try {
              update((int) ((currentUpdateTime - lastUpdateTime)/(1000*1000)));
          } catch (IOException ex) {
              Logger.getLogger(Game.class.getName()).log(Level.SEVERE, null, ex);
          }
         
         endLoopTime = System.nanoTime();
         deltaLoop = endLoopTime - beginLoopTime;
           
           if(deltaLoop <= desiredDeltaLoop){
               try{
                   Thread.sleep((desiredDeltaLoop - deltaLoop)/(1000*1000));
               }catch(InterruptedException e){}
           }
      }
   }
   
   private void render() {
      Graphics2D g = (Graphics2D) bufferStrategy.getDrawGraphics();
      g.clearRect(0, 0, WIDTH, HEIGHT);
      render(g);
      g.dispose();
      bufferStrategy.show();
   }
  
   private int gameState;
   private double splash;
   private double timer;
   private int animate = 1;
   private Image background, farmer, wolf, sheep, cabbage, boat, carry;
   private int fX = 5, fY = 380, wX = 110, wY = 330, sX = 90, sY = 390, cX = 35, cY = 320, bX = 130, bY = 480;
   private int x, y, x1, x2, y1, y2;
   private boolean charDragged = false;
        private int selected = -1; //farmer, wolf, sheep, cabbage
        private boolean intoBoat = false;
        private boolean outOfBoat = false;
   private boolean[] isRight = new boolean[5];
   private boolean[] isInBoat = new boolean[4];
   private long startTime;
   private boolean playing = false;
   
   private boolean crossing = false;
   private int moves = 0;
   private final int fW = 100, fH = 120, wW = 75,wH = 60, sW = 75, sH = 60, cW = 50, cH = 50, bW = 200, bH = 60;
   private int gameEndScreen;
           
   

   protected void update(int deltaTime) throws IOException{
       File tempFile;
        if(gameState == 0){
            tempFile = new File("test\\img\\splashScreen.png");
        } else if(gameState == 2){
            tempFile = new File("test\\img\\gameBackground.png");
        } else if(gameState == 3){
            if(gameEndScreen == 0){
                tempFile = new File("test\\img\\wolfSheepCabbageBadEnd.png");
            } else if(gameEndScreen == 1){
                tempFile = new File("test\\img\\sheepCabbageBadEnd.png");
            }else if(gameEndScreen == 2){
                tempFile = new File("test\\img\\wolfSheepBadEnd.png");
            }else if(gameEndScreen == 3){
                tempFile = new File("test\\img\\goodEnd.png");
            } else {
            tempFile = new File("");
            }
        } else {
            tempFile = new File("");
        }
        
        background = ImageIO.read(tempFile);
        
        
        //Splashscreen
        if(gameState == 0){
            if(splash < 500){
                splash += deltaTime * 0.2;
            }else{
               gameState = 2;
            }
        } 

        //Gameplay
        else if(gameState == 2){
            if(playing == false){
                startTime = System.nanoTime();
                playing = true;
            }
            timer += deltaTime * 0.2;
            //AnimationTimer
            while(timer > 50){
                timer -= 50;
                if(animate < 3){
                    animate++;
                } else {
                    animate = 1;
                }
                //CharacterFiles
                for(int i = 0; i < 5; i++){
                    if(isRight[i] == false){
                        if(i == 0){
                            //farmerLeft
                            tempFile = new File("test\\img\\farmer\\L" + animate + ".png");
                            farmer = ImageIO.read(tempFile);
                        } else if(i == 1){
                            //wolfLeft
                            tempFile = new File("test\\img\\wolf\\L" + animate + ".png");
                            wolf = ImageIO.read(tempFile);
                        } else if(i == 2){
                            //sheepLeft
                            tempFile = new File("test\\img\\sheep\\L" + animate + ".png");
                            sheep = ImageIO.read(tempFile);
                        } else if(i == 3){
                            //cabbageLeft
                            tempFile = new File("test\\img\\cabbage\\L" + animate + ".png");
                            cabbage = ImageIO.read(tempFile);
                        } else if(i == 4){
                            //boatLeft
                            tempFile = new File("test\\img\\boat\\" + animate + ".png");
                            boat = ImageIO.read(tempFile);
                        }
                    } else {
                        if(i == 0){
                            //farmerRight
                            tempFile = new File("test\\img\\farmer\\R" + animate + ".png");
                            farmer = ImageIO.read(tempFile);
                        } else if(i == 1){
                            //wolfRight
                            tempFile = new File("\test\\img\\wolf\\R" + animate + ".png");
                            wolf = ImageIO.read(tempFile);
                        } else if(i == 2){
                            //sheepRight
                            tempFile = new File("test\\img\\sheep\\R" + animate + ".png");
                            sheep = ImageIO.read(tempFile);
                        } else if(i == 3){
                            //cabbageRight
                            tempFile = new File("test\\img\\cabbage\\R" + animate + ".png");
                            cabbage = ImageIO.read(tempFile);
                        } else if(i == 4){
                            //boatRight
                            tempFile = new File("test\\img\\boat\\" + animate + ".png");
                            boat = ImageIO.read(tempFile);
                        }
                    }
                }
            }
            if(intoBoat == true){
                if(selected == 0){
                    fX = bX + 30;
                    fY = bY - (fH - bH + 20);
                }else if(selected == 1){
                    wX = bX + 115;
                    wY = bY - (wH - bH + 20);
                }else if(selected == 2){
                    sX = bX + 60;
                    sY = bY - (sH - bH + 20);
                }else if(selected == 3){
                    cX = bX + 5;
                    cY = bY - (cH - bH + 20);
                }
                isInBoat[selected] = true;
                intoBoat = false;
                selected = -1;
            }
            if(outOfBoat == true){
                if(isRight[selected] == false){
                    if(selected == 0){
                        fX = 5;
                        fY = 380;
                    }else if(selected == 1){
                        wX = 110;
                        wY = 330;
                    }else if(selected == 2){
                        sX = 90;
                        sY = 390;
                    }else if(selected == 3){
                        cX = 35;
                        cY = 320;
                    }
                } else {
                    if(selected == 0){
                        fX = 590;
                        fY = 380;
                    } else if (selected == 1){
                        wX = 670;
                        wY = 330;
                    } else if (selected == 2){
                        sX = 680;
                        sY = 390;
                    } else if (selected == 3){
                        cX = 600;
                        cY = 320;
                    }
                }
                isInBoat[selected] = false;
                outOfBoat = false;
                selected = -1;
            }
            if(crossing == true){
                if(isInBoat[0] == true){
                    if(isRight[4] == false){
                        //Left to right animation
                        if(bX < 360){
                            for(int i = 0; i < 4; i++){
                                if(isInBoat[i] == true){
                                    if(i == 0){
                                        fX += 2;
                                    } else if (i == 1){
                                        wX += 2;
                                    } else if (i == 2){
                                        sX += 2;
                                    } else if (i == 3){
                                        cX += 2;
                                    }
                                }
                            }
                            bX += 2;
                        } else {
                            for(int i = 0; i < 4; i++){
                                if(isInBoat[i] == true){
                                    if(i == 0){
                                        fX = 590;
                                        fY = 380;
                                    } else if (i == 1){
                                        wX = 670;
                                        wY = 330;
                                    } else if (i == 2){
                                        sX = 680;
                                        sY = 390;
                                    } else if (i == 3){
                                        cX = 600;
                                        cY = 320;
                                    }
                                    isRight[i] = true;
                                    isInBoat[i] = false;
                                }
                                isRight[4] = true;
                            }
                            moves ++;
                            crossing = false;
                            //Test if game over
                            if((isRight[1] || isRight[2] || isRight[3]) == false){
                                //All left
                                gameState = 3;
                                gameEndScreen = 0;
                                //Wolf ate everything
                            } else if((isRight[2] || isRight[3]) == false){
                                //Cabbage & Sheep
                                gameState = 3;
                                gameEndScreen = 1;
                                //Sheep ate Cabbage
                            } else if((isRight[1] || isRight[2]) == false){
                                //Wolf & Sheep
                                gameState = 3;
                                gameEndScreen = 2;
                                //Wolf ate Sheep
                            } else if(isRight[1] && isRight[2] && isRight[3]){
                                //All right
                                gameState = 3;
                                gameEndScreen = 3;
                                //Win!
                            }
                                
                        }
                    } else {
                        //Right to left animation
                        if(bX > 130){
                            for(int i = 0; i < 4; i++){
                                if(isInBoat[i] == true){
                                    if(i == 0){
                                        fX -= 2;
                                    } else if (i == 1){
                                        wX -= 2;
                                    } else if (i == 2){
                                        sX -= 2;
                                    } else if (i == 3){
                                        cX -= 2;
                                    }
                                }
                            } 
                            bX -= 2;
                        } else {
                            for(int i = 0; i < 4; i++){
                                if(isInBoat[i] == true){
                                    if(i == 0){
                                        fX = 5;
                                        fY = 380;
                                    } else if (i == 1){
                                        wX = 110;
                                        wY = 330;
                                    } else if (i == 2){
                                        sX = 90;
                                        sY = 390;
                                    } else if (i == 3){
                                        cX = 35;
                                        cY = 320;
                                    }
                                    isRight[i] = false;
                                    isInBoat[i] = false;
                                }
                                isRight[4] = false;
                            }
                            moves++;
                            crossing = false;
                            //Test if game over
                            if(isRight[1] && isRight[2] && isRight[3]){
                                //All Right
                                gameState = 3;
                                gameEndScreen = 0;
                                //Wolf ate everything
                            } else if(isRight[2] && isRight[3]){
                                //Cabbage & Sheep
                                gameState = 3;
                                gameEndScreen = 1;
                                //Sheep ate Cabbage
                            } else if(isRight[1] && isRight[2]){
                                //Wolf & Sheep
                                gameState = 3;
                                gameEndScreen = 2;
                                //Wolf ate Sheep
                            }
                                
                        }
                    }
                } else {
                    //Farmer not in boat - error
                    crossing = false;
                }
         
            }
        }
   }
   

   protected void render(Graphics2D g){
       Font font = new Font("Serif", Font.PLAIN, 25);
       g.setFont(font);
       g.setColor(Color.WHITE);
       try {
            g.drawImage(background, 0, 0, null);
            if(gameState == 2){
                if(charDragged == true){
                    if(selected == 0){
                        g.drawImage(wolf, wX, wY, null);
                        g.drawImage(sheep, sX, sY, null);
                        g.drawImage(cabbage, cX, cY, null);
                        g.drawImage(carry, x, y, null);
                    }if(selected == 1){
                        g.drawImage(farmer, fX, fY, null);
                        g.drawImage(sheep, sX, sY, null);
                        g.drawImage(cabbage, cX, cY, null);
                        g.drawImage(carry, x, y, null);
                    }if(selected == 2){
                        g.drawImage(farmer, fX, fY, null);
                        g.drawImage(wolf, wX, wY, null);
                        g.drawImage(cabbage, cX, cY, null);
                        g.drawImage(carry, x, y, null);
                    }if(selected == 3){
                        g.drawImage(farmer, fX, fY, null);
                        g.drawImage(wolf, wX, wY, null);
                        g.drawImage(sheep, sX, sY, null);
                        g.drawImage(carry, x, y, null);
                    }
                }else{
                    g.drawImage(farmer, fX, fY, null);
                    g.drawImage(wolf, wX, wY, null);
                    g.drawImage(sheep, sX, sY, null);
                    g.drawImage(cabbage, cX, cY, null);
                }
                g.drawImage(boat, bX, bY, null);
                g.drawString("Time Elapsed: " + Integer.toString((int)(System.nanoTime()/(1000*1000*1000) - startTime/(1000*1000*1000))), 20, 32);
                g.drawString("Number of moves: " + Integer.toString(moves), 520, 32);
            }
       }catch(Exception ex){
           System.out.println(ex);
       }
   }
   
   public static void main(String [] args){
      Game ex = new Game();
      new Thread(ex).start();
   }

}
