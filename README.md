import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;

public class PlatformerGame extends JFrame {
    
    public PlatformerGame() {
        setTitle("Jogo de Plataforma Simples");
        setSize(800, 400);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        setResizable(false);
        add(new GamePanel());
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            new PlatformerGame().setVisible(true);
        });
    }
}

class GamePanel extends JPanel implements ActionListener, KeyListener {

    private final int GROUND_Y = 350;
    private final int PLAYER_SIZE = 30;
    private int playerX = 100, playerY = GROUND_Y - PLAYER_SIZE;
    private int velocityX = 0, velocityY = 0;
    private boolean isJumping = false;
    private boolean isOnGround = false;
    private Timer timer;

    private ArrayList<Rectangle> platforms;
    private Image playerImage;
    private Image backgroundImage;
    private Image itemImage;

    private Rectangle itemRect;
    private boolean itemCollected = false;

    public GamePanel() {
        setFocusable(true);
        addKeyListener(this);

        playerImage = new ImageIcon("player.jpg").getImage();
        backgroundImage = new ImageIcon("background.jpg").getImage();
        itemImage = new ImageIcon("item.png").getImage();

        platforms = new ArrayList<>();
        platforms.add(new Rectangle(100, 300, 200, 20));
        platforms.add(new Rectangle(400, 250, 200, 20));
        platforms.add(new Rectangle(200, 150, 200, 20));

        itemRect = new Rectangle(450, 220, 20, 20); // Item sobre uma plataforma

        timer = new Timer(10, this);
        timer.start();
    }

    @Override
    public void paintComponent(Graphics g) {
        super.paintComponent(g);

        g.drawImage(backgroundImage, 0, 0, getWidth(), getHeight(), this);

        g.setColor(Color.BLACK);
        for (Rectangle platform : platforms) {
            g.fillRect(platform.x, platform.y, platform.width, platform.height);
        }

        if (!itemCollected) {
            g.drawImage(itemImage, itemRect.x, itemRect.y, itemRect.width, itemRect.height, this);
        }

        g.drawImage(playerImage, playerX, playerY, PLAYER_SIZE, PLAYER_SIZE, this);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        if (isJumping) {
            velocityY += 1;
            playerY += velocityY;
            isOnGround = false;

            if (playerY >= GROUND_Y - PLAYER_SIZE) {
                playerY = GROUND_Y - PLAYER_SIZE;
                isJumping = false;
                velocityY = 0;
                isOnGround = true;
            }
        }

        for (Rectangle platform : platforms) {
            if (new Rectangle(playerX, playerY + PLAYER_SIZE, PLAYER_SIZE, 1).intersects(platform)) {
                playerY = platform.y - PLAYER_SIZE;
                isJumping = false;
                velocityY = 0;
                isOnGround = true;
                break;
            }
        }

        // Movimento
        playerX += velocityX;
        if (playerX < 0) playerX = 0;
        if (playerX > getWidth() - PLAYER_SIZE) playerX = getWidth() - PLAYER_SIZE;

        // Coletar item automaticamente ao encostar
        if (!itemCollected) {
            Rectangle playerRect = new Rectangle(playerX, playerY, PLAYER_SIZE, PLAYER_SIZE);
            if (playerRect.intersects(itemRect)) {
                itemCollected = true;
                System.out.println("Item coletado automaticamente!");
            }
        }

        repaint();
    }

    @Override
    public void keyPressed(KeyEvent e) {
        int key = e.getKeyCode();

        if (key == KeyEvent.VK_LEFT) {
            velocityX = -5;
        }
        if (key == KeyEvent.VK_RIGHT) {
            velocityX = 5;
        }
        if (key == KeyEvent.VK_SPACE && isOnGround) {
            isJumping = true;
            isOnGround = false;
            velocityY = -15;
        }
    }

    @Override
    public void keyReleased(KeyEvent e) {
        int key = e.getKeyCode();
        if (key == KeyEvent.VK_LEFT || key == KeyEvent.VK_RIGHT) {
            velocityX = 0;
        }
    }

    @Override
    public void keyTyped(KeyEvent e) {}
}
