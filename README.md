import java.awt.event.KeyEvent;
import java.awt.event.KeyListener;
import java.util.ArrayList;
import java.util.List;


class GameBoard extends JPanel implements ActionListener, KeyListener {


    abstract class GameObject {
        private int x, y, width, height;

        GameObject(int x, int y, int width, int height) {
            this.x = x;
            this.y = y;
            this.width = width;
            this.height = height;
        }


        abstract void draw(Graphics g);

        public Rectangle getBounds() {
            return new Rectangle(x, y, width, height);
        }


        public int getX() { return x; }
        public void setX(int x) { this.x = x; }
        public int getY() { return y; }
        public void setY(int y) { this.y = y; }
        public int getWidth() { return width; }
        public void setWidth(int width) { this.width = width; }
        public int getHeight() { return height; }
        public void setHeight(int height) { this.height = height; }
    }


    class Paddle extends GameObject {
        private int speed = 8;

        Paddle(int x, int y) {
            super(x, y, 80, 15);
        }

        void moveLeft() {
            setX(getX() - speed);
            if (getX() < 0) setX(0);
        }

        void moveRight(int boardWidth) {
            setX(getX() + speed);
            if (getX() + getWidth() > boardWidth) setX(boardWidth - getWidth());
        }

        @Override
        void draw(Graphics g) {
            g.setColor(Color.WHITE);
            g.fillRect(getX(), getY(), getWidth(), getHeight());
        }

        public int getSpeed() { return speed; }
        public void setSpeed(int speed) { this.speed = speed; }
    }

    class Ball extends GameObject {
        private int dx = 4;
        private int dy = -4;

        Ball(int x, int y) {
            super(x, y, 15, 15);
        }

        void update() {
            setX(getX() + dx);
            setY(getY() + dy);
        }


        void draw(Graphics g) {
            g.setColor(Color.WHITE);
            g.fillOval(getX(), getY(), getWidth(), getHeight());
        }

        public int getDx() { return dx; }
        public void setDx(int dx) { this.dx = dx; }
        public int getDy() { return dy; }
        public void setDy(int dy) { this.dy = dy; }
    }
    
    class Brick extends GameObject {
        private boolean destroyed = false;
        private Color color;

        Brick(int x, int y) {
            super(x, y, 60, 20);
            color = new Color(
                    (int)(Math.random() * 255),
                    (int)(Math.random() * 255),
                    (int)(Math.random() * 255)
            );
        }


        void draw(Graphics g) {
            if (!destroyed) {
                g.setColor(color);
                g.fillRect(getX(), getY(), getWidth(), getHeight());
            }
        }

        public boolean isDestroyed() { return destroyed; }
        public void setDestroyed(boolean destroyed) { this.destroyed = destroyed; }
        public Color getColor() { return color; }
    }


    private Paddle paddle;
    private Ball ball;
    private List<Brick> bricks;
    private List<GameObject> drawables;
    private Timer timer;
    private boolean running = true;
    private String message = "";
    private Font customFont;

    GameBoard() {
        setBackground(Color.BLACK);
        setFocusable(true);
        addKeyListener(this);

               try {
           customFont = new Font("Poppins", Font.BOLD, 30);
       } catch (Exception e) {
           customFont = new Font("SansSerif", Font.BOLD, 30);
           System.err.println("Error initializing custom font: " + e.getMessage());
       }


       initGameObjects();


       timer = new Timer(16, this);
       timer.start();
   }


   private void initGameObjects() {
       paddle = new Paddle(260, 450);
       ball = new Ball(295, 300);
       createBricks();




       drawables = new ArrayList<>();
       drawables.add(paddle);
       drawables.add(ball);
       drawables.addAll(bricks);
   }


   void createBricks() {
       bricks = new ArrayList<>();
       int rows = 5, cols = 8;


       for (int r = 0; r < rows; r++) {
           for (int c = 0; c < cols; c++) {
               bricks.add(new Brick(20 + c * 70, 30 + r * 30));
           }
       }
   }


   void restartGame() {
       initGameObjects();
       message = "";
       running = true;
       timer.start();
       repaint();
   }




   public void actionPerformed(ActionEvent e) {
       if (!running) return;
       ball.update();
       checkCollisions();
       repaint();
   }
   
void checkCollisions() {
        if (ball.getX() <= 0 || ball.getX() + ball.getWidth() >= getWidth())
            ball.setDx(ball.getDx() * -1);


        if (ball.getY() <= 0)
            ball.setDy(ball.getDy() * -1);


        if (ball.getY() + ball.getHeight() >= getHeight()) {
            running = false;
            timer.stop();
            message = "Game Over!";
        }


        if (ball.getBounds().intersects(paddle.getBounds())) {
            ball.setDy(ball.getDy() * -1);
            ball.setY(paddle.getY() - ball.getHeight()); // Fixed Subtraction
        }


        boolean allDestroyed = true;


        for (Brick b : bricks) {
            if (!b.isDestroyed()) {
                allDestroyed = false;


                if (ball.getBounds().intersects(b.getBounds())) {
                    b.setDestroyed(true);
                    ball.setDy(ball.getDy() * -1);
                }
            }
        }


        if (allDestroyed) {
            running = false;
            timer.stop();
            message = "You Win!";
        }
    }




    protected void paintComponent(Graphics g) {
        super.paintComponent(g);


        // 4. Polymorphism in Execution
        for (GameObject obj : drawables) {
            obj.draw(g);
        }


        if (!running) {
            g.setColor(Color.WHITE);
            g.setFont(customFont);
            g.drawString(message, getWidth() / 2 - 70, getHeight() / 2);


            g.setFont(new Font("SansSerif", Font.PLAIN, 18));
            g.drawString("Press SPACE to play again",
                    getWidth() / 2 - 110, getHeight() / 2 + 40); // Fixed Subtraction
        }
    }




    public void keyPressed(KeyEvent e) {
        if (running) {
            if (e.getKeyCode() == KeyEvent.VK_LEFT)
                paddle.moveLeft();


            if (e.getKeyCode() == KeyEvent.VK_RIGHT)
                paddle.moveRight(getWidth());
        }


        if (!running && e.getKeyCode() == KeyEvent.VK_SPACE) {
            restartGame();
        }
    }


    public void keyReleased(KeyEvent e) {}
    public void keyTyped(KeyEvent e) {}
}




public class BrickBreaker extends JFrame {


    public BrickBreaker() {
        setTitle("Brick Breaker");
        setSize(600, 500);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);


        add(new GameBoard());
        setVisible(true);
    }


    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new BrickBreaker());
    }
}
