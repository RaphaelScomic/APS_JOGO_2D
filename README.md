import javax.swing.*;
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;

public class PequenasAcoes extends JPanel implements ActionListener, KeyListener {

    private Timer temporizador;
    private Timer temporizadorFinal;
    private int posicaoX = 50, posicaoY = 300;
    private int velocidadeX = 0, velocidadeY = 0;
    private boolean estaNoChao = true;
    private final int GRAVIDADE = 1;
    private boolean jogoFinalizado = false;
    private boolean tempoEsgotado = false;
    private long tempoInicial;
    private int tempoRestante;

    private Rectangle plataforma1 = new Rectangle(300, 250, 200, 20);
    private Rectangle plataforma2 = new Rectangle(600, 100, 150, 20);
    private ArrayList<Rectangle> itensColetaveis = new ArrayList<>();

    public PequenasAcoes() {
        JFrame janela = new JFrame("Jogo Plataforma 2D");
        janela.setSize(800, 600);
        janela.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        janela.add(this);
        janela.setVisible(true);
        janela.addKeyListener(this);

        itensColetaveis.add(new Rectangle(100, 385, 15, 15));
        itensColetaveis.add(new Rectangle(200, 385, 15, 15));
        itensColetaveis.add(new Rectangle(400, 235, 15, 15));
        itensColetaveis.add(new Rectangle(500, 385, 15, 15));
        itensColetaveis.add(new Rectangle(650, 385, 15, 15));
        itensColetaveis.add(new Rectangle(610, 85, 15, 15));
        itensColetaveis.add(new Rectangle(690, 85, 15, 15));

        tempoInicial = System.currentTimeMillis();

        temporizador = new Timer(15, this);
        temporizador.start();

        temporizadorFinal = new Timer(10000, new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                tempoEsgotado = true;
                temporizador.stop();
                temporizadorFinal.stop();
                repaint();
            }
        });
        temporizadorFinal.setRepeats(false);
        temporizadorFinal.start();
    }

    @Override
    public void paintComponent(Graphics g) {
        super.paintComponent(g);

        if (jogoFinalizado) {
            g.setColor(PRETO);
            g.fillRect(0, 0, getWidth(), getHeight());
            g.setColor(BRANCO);
            g.setFont(new Font("Arial", Font.BOLD, 30));
            String mensagem = "Parabéns, você deixou o mundo mais limpo!";
            FontMetrics fm = g.getFontMetrics();
            int x = (getWidth() - fm.stringWidth(mensagem)) / 2;
            int y = getHeight() / 2;
            g.drawString(mensagem, x, y);

        } else if (tempoEsgotado) {
            g.setColor(PRETO);
            g.fillRect(0, 0, getWidth(), getHeight());
            g.setColor(VERMELHO);
            g.setFont(new Font("Arial", Font.BOLD, 24));

            String linha1 = "O lixo espalhado acarretou em uma cidade mais suja, que por consequência";
            String linha2 = "gerou inúmeras doenças. GAME OVER :(";

            FontMetrics fm = g.getFontMetrics();
            int y = getHeight() / 2;

            int x1 = (getWidth() - fm.stringWidth(linha1)) / 2;
            int x2 = (getWidth() - fm.stringWidth(linha2)) / 2;

            g.drawString(linha1, x1, y);
            g.drawString(linha2, x2, y + 30);

        } else {
            // Fundo azul claro
            g.setColor(AZUL_CLARO);
            g.fillRect(0, 0, getWidth(), getHeight());

            g.setColor(CINZA);
            g.fillRect(0, 400, getWidth(), 200);

            g.setColor(CINZA_ESCURO);
            g.fillRect(plataforma1.x, plataforma1.y, plataforma1.width, plataforma1.height);
            g.fillRect(plataforma2.x, plataforma2.y, plataforma2.width, plataforma2.height);

            // Itens verdes
            g.setColor(VERDE);
            for (Rectangle item : itensColetaveis) {
                g.fillRect(item.x, item.y, item.width, item.height);
            }

            // Personagem vermelho
            g.setColor(VERMELHO);
            g.fillRect(posicaoX, posicaoY, 50, 50);

            g.setColor(PRETO);
            g.setFont(new Font("Arial", Font.BOLD, 30));
            String titulo = "Colete os Lixos";
            FontMetrics fm = g.getFontMetrics();
            g.drawString(titulo, (getWidth() - fm.stringWidth(titulo)) / 2, 50);

            tempoRestante = 10 - (int) ((System.currentTimeMillis() - tempoInicial) / 1000);
            g.setFont(new Font("Arial", Font.BOLD, 20));
            g.drawString("Tempo: " + tempoRestante + "s", getWidth() - 150, 30);
        }
    }

    @Override
    public void actionPerformed(ActionEvent evento) {
        posicaoX += velocidadeX;
        posicaoY += velocidadeY;

        Rectangle personagem = new Rectangle(posicaoX, posicaoY, 50, 50);

        velocidadeY += GRAVIDADE;
        estaNoChao = false;

        estaNoChao = verificaColisaoComPlataforma(personagem, plataforma1)
                || verificaColisaoComPlataforma(personagem, plataforma2);

        if (posicaoY >= 350) {
            posicaoY = 350;
            velocidadeY = 0;
            estaNoChao = true;
        }

        itensColetaveis.removeIf(personagem::intersects);

        if (itensColetaveis.isEmpty()) {
            jogoFinalizado = true;
            temporizador.stop();
            temporizadorFinal.stop();
        }

        repaint();
    }

    private boolean verificaColisaoComPlataforma(Rectangle personagem, Rectangle plataforma) {
        if (personagem.intersects(plataforma)) {
            Rectangle intersecao = personagem.intersection(plataforma);
            if (intersecao.height < intersecao.width &&
                velocidadeY >= 0 &&
                personagem.y + personagem.height <= plataforma.y + velocidadeY) {

                posicaoY = plataforma.y - 50;
                velocidadeY = 0;
                return true;
            }
        }
        return false;
    }

    @Override
    public void keyPressed(KeyEvent evento) {
        int tecla = evento.getKeyCode();

        if (tecla == KeyEvent.VK_LEFT) {
            velocidadeX = -5;
        } else if (tecla == KeyEvent.VK_RIGHT) {
            velocidadeX = 5;
        } else if (tecla == KeyEvent.VK_SPACE && estaNoChao) {
            velocidadeY = -20;
            estaNoChao = false;
        }
    }

    @Override
    public void keyReleased(KeyEvent evento) {
        int tecla = evento.getKeyCode();
        if (tecla == KeyEvent.VK_LEFT || tecla == KeyEvent.VK_RIGHT) {
            velocidadeX = 0;
        }
    }

    @Override
    public void keyTyped(KeyEvent evento) {
    }

    public static void main(String[] args) {
        new PequenasAcoes();
    }

    private static final Color PRETO = Color.BLACK;
    private static final Color BRANCO = Color.WHITE;
    private static final Color VERMELHO = Color.RED;
    private static final Color VERDE = Color.GREEN;
    private static final Color CINZA = Color.GRAY;
    private static final Color CINZA_ESCURO = Color.DARK_GRAY;
    private static final Color AZUL_CLARO = new Color(173, 216, 230); // Light blue
}
