import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;
import javax.swing.*;

public class PequenasAcoes extends JPanel implements ActionListener, KeyListener {

    private Timer temporizador;
    private int posicaoX = 50, posicaoY = 300; // Posição inicial do personagem
    private int velocidadeX = 0, velocidadeY = 0; // Velocidade do personagem
    private boolean noChao = true; // Verifica se o personagem está no chão
    private final int GRAVIDADE = 1;

    private Rectangle plataforma1 = new Rectangle(300, 250, 200, 20); // Plataforma 1
    private Rectangle plataforma2 = new Rectangle(600, 100, 150, 20); // Plataforma 2
    private ArrayList<Rectangle> itens = new ArrayList<>(); // Lista de itens
    private boolean fimDeJogo = false; // Controla o estado do jogo (fim ou não)

    public PequenasAcoes() {
        JFrame frame = new JFrame("Jogo Plataforma 2D");
        frame.setSize(800, 600);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.add(this);
        frame.setVisible(true);
        frame.addKeyListener(this);

        // Itens no chão e na plataforma 1
        itens.add(new Rectangle(100, 385, 15, 15));
        itens.add(new Rectangle(200, 385, 15, 15));
        itens.add(new Rectangle(400, 235, 15, 15)); // Em cima da plataforma 1
        itens.add(new Rectangle(500, 385, 15, 15));
        itens.add(new Rectangle(650, 385, 15, 15));

        // Novos itens em cima da plataforma 2 (superior direita)
        itens.add(new Rectangle(610, 85, 15, 15)); // Esquerda da plataforma 2
        itens.add(new Rectangle(690, 85, 15, 15)); // Centro da plataforma 2

        temporizador = new Timer(15, this); // Atualiza a tela a cada 15ms
        temporizador.start();
    }

    @Override
    public void paintComponent(Graphics g) {
        super.paintComponent(g);

        // Se o jogo terminou, exibe a tela preta com a mensagem
        if (fimDeJogo) {
            g.setColor(Color.BLACK);
            g.fillRect(0, 0, getWidth(), getHeight()); // Tela preta

            // Mensagem centralizada
            g.setColor(Color.WHITE);
            g.setFont(new Font("Arial", Font.BOLD, 30)); // Fonte da mensagem
            String mensagem = "Parabéns, você deixou o mundo mais limpo!";
            FontMetrics metrics = g.getFontMetrics();
            int mensagemX = (getWidth() - metrics.stringWidth(mensagem)) / 2;
            int mensagemY = getHeight() / 2; // Centraliza verticalmente
            g.drawString(mensagem, mensagemX, mensagemY);
        } else {
            // Fundo do jogo
            g.setColor(Color.CYAN);
            g.fillRect(0, 0, getWidth(), getHeight());

            // Chão
            g.setColor(Color.GREEN);
            g.fillRect(0, 400, getWidth(), 200);

            // Plataforma 1 (meio)
            g.setColor(Color.DARK_GRAY);
            g.fillRect(plataforma1.x, plataforma1.y, plataforma1.width, plataforma1.height);

            // Plataforma 2 (superior direita)
            g.fillRect(plataforma2.x, plataforma2.y, plataforma2.width, plataforma2.height);

            // Itens coletáveis
            g.setColor(Color.BLACK);
            for (Rectangle item : itens) {
                g.fillRect(item.x, item.y, item.width, item.height);
            }

            // Personagem (quadrado vermelho)
            g.setColor(Color.RED);
            g.fillRect(posicaoX, posicaoY, 50, 50);

            // Título: "Colete os Lixos"
            g.setColor(Color.BLACK);
            g.setFont(new Font("Arial", Font.BOLD, 30)); // Fonte do título
            String titulo = "Colete os Lixos";
            FontMetrics metrics = g.getFontMetrics();
            int tituloX = (getWidth() - metrics.stringWidth(titulo)) / 2;
            int tituloY = 50;
            g.drawString(titulo, tituloX, tituloY); // Desenha o título na tela
        }
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        posicaoX += velocidadeX;
        posicaoY += velocidadeY;

        Rectangle personagem = new Rectangle(posicaoX, posicaoY, 50, 50);

        // Gravidade
        velocidadeY += GRAVIDADE;
        noChao = false;

        // Colisão com plataformas
        noChao = verificaColisaoComPlataforma(personagem, plataforma1) || verificaColisaoComPlataforma(personagem, plataforma2);

        // Colisão com o chão
        if (posicaoY >= 350) {
            posicaoY = 350;
            velocidadeY = 0;
            noChao = true;
        }

        // Coleta de itens (se o personagem intersectar os itens)
        itens.removeIf(personagem::intersects);

        // Verifica se todos os itens foram coletados
        if (itens.isEmpty()) {
            fimDeJogo = true; // Ativa o fim de jogo
        }

        repaint();
    }

    private boolean verificaColisaoComPlataforma(Rectangle personagem, Rectangle plataforma) {
        if (personagem.intersects(plataforma)) {
            Rectangle intersecao = personagem.intersection(plataforma);
            if (intersecao.height < intersecao.width && velocidadeY >= 0 && personagem.y + personagem.height <= plataforma.y + velocidadeY) {
                posicaoY = plataforma.y - 50;
                velocidadeY = 0;
                return true;
            }
        }
        return false;
    }

    @Override
    public void keyPressed(KeyEvent e) {
        int tecla = e.getKeyCode();
        if (tecla == KeyEvent.VK_LEFT) {
            velocidadeX = -5; // Move para a esquerda
        } else if (tecla == KeyEvent.VK_RIGHT) {
            velocidadeX = 5; // Move para a direita
        } else if (tecla == KeyEvent.VK_SPACE && noChao) {
            velocidadeY = -20; // Pulo
            noChao = false;
        }
    }

    @Override
    public void keyReleased(KeyEvent e) {
        int tecla = e.getKeyCode();
        if (tecla == KeyEvent.VK_LEFT || tecla == KeyEvent.VK_RIGHT) {
            velocidadeX = 0; // Para de mover o personagem horizontalmente
        }
    }

    @Override
    public void keyTyped(KeyEvent e) {}

    public static void main(String[] args) {
        new PequenasAcoes();
    }
}
