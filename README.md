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
