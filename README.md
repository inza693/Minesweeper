# Minesweeper
MINE SWEEPER (simple class)
import java.awt.*;
import java.awt.event.*;
import java.util.ArrayList;
import java.util.Random;
import javax.swing.*;
public class Minesweeper {
    // Change this from private to public or protected
    public class MineTile extends JButton {
        // Class implementation remains the same

    // Rest of the Minesweeper class..
        int r, c;
        boolean isMine = false;  // Add this to track if the tile contains a mine
        private int adjacentMines = -1;  // Store the number of adjacent mines

        public MineTile(int r, int c) {
            this.r = r;
            this.c = c;
        }

        public void setMine(boolean isMine) {
            this.isMine = isMine;
        }

        public boolean isMine() {
            return this.isMine;
        }

        public void setAdjacentMines(int adjacentMines) {
            this.adjacentMines = adjacentMines;
        }

        public int getAdjacentMines() {
            return this.adjacentMines;
        }

        // Override the paintComponent to ensure the numbers are painted with the correct colors
        @Override
        protected void paintComponent(Graphics g) {
            super.paintComponent(g);
            if (adjacentMines > 0) {
                // Set font size
                Font font = new Font("Arial", Font.BOLD, 24);
                g.setFont(font);
                // Set color based on the number of adjacent mines
                switch (adjacentMines) {
                    case 1:
                        g.setColor(new Color(139, 69, 19));  // Brown for 1
                        break;
                    case 2:
                        g.setColor(new Color(0, 128, 0));  // Green for 2
                        break;
                    case 3:
                        g.setColor(new Color(255, 105, 180));  // Pink for 3
                        break;
                    default:
                        g.setColor(Color.BLACK);  // Default black for other numbers
                        break;
                }
                String text = String.valueOf(adjacentMines);
                FontMetrics fm = g.getFontMetrics();
                int x = (getWidth() - fm.stringWidth(text)) / 2;
                int y = (getHeight() + fm.getAscent()) / 2;
                g.drawString(text, x, y);
            }
        }
    }

    int tileSize = 70;
    int numRows = 8;
    int numCols = numRows;
    int boardWidth = numCols * tileSize;
    int boardHeight = numRows * tileSize;

    JFrame frame = new JFrame("Minesweeper");
    JLabel textLabel = new JLabel();
    JPanel textPanel = new JPanel();
    JPanel boardPanel = new JPanel();

    int mineCount = 10;
    MineTile[][] board = new MineTile[numRows][numCols];
    ArrayList<MineTile> mineList;
    Random random = new Random();

    int tilesClicked = 0; // goal is to click all tiles except the ones containing mines
    boolean gameOver = false;

    public Minesweeper() {
        showWelcomeScreen();
    }

    private void showWelcomeScreen() {
        JFrame welcomeFrame = new JFrame("Welcome to Minesweeper");
        welcomeFrame.setSize(400, 300);
        welcomeFrame.setLocationRelativeTo(null);
        welcomeFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        welcomeFrame.setLayout(new BorderLayout());

        JPanel panel = new JPanel();
        panel.setBackground(Color.BLACK); // Black background
        panel.setLayout(new BorderLayout());

        JLabel welcomeLabel = new JLabel("Welcome to the game!", SwingConstants.CENTER);
        welcomeLabel.setFont(new Font("Arial", Font.BOLD, 24));
        welcomeLabel.setForeground(Color.WHITE); // White text
        panel.add(welcomeLabel, BorderLayout.NORTH);

        JPanel buttonPanel = new JPanel();
        buttonPanel.setBackground(Color.BLACK); // Ensure button panel matches background
        buttonPanel.setLayout(new FlowLayout()); // Center buttons

        JButton startButton = new JButton("Start Game");
        startButton.setFont(new Font("Arial", Font.PLAIN, 18));
        startButton.setBackground(new Color(204, 153, 255)); // Light purple
        startButton.setFocusPainted(false);
        startButton.setBorder(BorderFactory.createRaisedBevelBorder());

        JButton howToPlayButton = new JButton("How to Play");
        howToPlayButton.setFont(new Font("Arial", Font.PLAIN, 18));
        howToPlayButton.setBackground(new Color(204, 153, 255)); // Light purple
        howToPlayButton.setFocusPainted(false);
        howToPlayButton.setBorder(BorderFactory.createRaisedBevelBorder());

        buttonPanel.add(startButton);
        buttonPanel.add(howToPlayButton);
        panel.add(buttonPanel, BorderLayout.CENTER);

        welcomeFrame.add(panel);
        welcomeFrame.setVisible(true);

        startButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                welcomeFrame.dispose(); // Close welcome screen
                initializeGame();      // Start the game
            }
        });

        howToPlayButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                showInstructions(); // Show how to play instructions
            }
        });
    }

    private void showInstructions() {
        JOptionPane.showMessageDialog(frame,
                "How to Play Minesweeper:\n\n" +
                        "1. Left-click on a tile to reveal it.\n" +
                        "2. If you click on a mine, the game is over.\n" +
                        "3. Numbers indicate how many mines are adjacent to the tile.\n" +
                        "4. Right-click to flag a tile as a mine.\n" +
                        "5. Clear all non-mine tiles to win!\n",
                "How to Play",
                JOptionPane.INFORMATION_MESSAGE);
    }

    private void initializeGame() {
        frame.setSize(boardWidth, boardHeight);
        frame.setLocationRelativeTo(null);
        frame.setResizable(false);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setLayout(new BorderLayout());

        textLabel.setFont(new Font("Arial", Font.BOLD, 25));
        textLabel.setHorizontalAlignment(JLabel.CENTER);
        textLabel.setText("Minesweeper: " + mineCount);
        textLabel.setOpaque(true);

        textPanel.setLayout(new BorderLayout());
        textPanel.add(textLabel);
        frame.add(textPanel, BorderLayout.NORTH);

        boardPanel.setLayout(new GridLayout(numRows, numCols)); // 8x8
        boardPanel.setBackground(new Color(220, 220, 255)); // Light lavender background
        frame.add(boardPanel);

        for (int r = 0; r < numRows; r++) {
            for (int c = 0; c < numCols; c++) {
                MineTile tile = new MineTile(r, c);
                board[r][c] = tile;

                tile.setFocusable(false);
                tile.setMargin(new Insets(0, 0, 0, 0));
                tile.setFont(new Font("Arial Unicode MS", Font.PLAIN, 10));
                tile.addMouseListener(new MouseAdapter() {
                    @Override
                    public void mousePressed(MouseEvent e) {
                        if (gameOver) {
                            return;
                        }
                        MineTile tile = (MineTile) e.getSource();

                        // Left click
                        if (e.getButton() == MouseEvent.BUTTON1) {
                            if (tile.getText().equals("")) {
                                if (mineList.contains(tile)) {
                                    revealMines();
                                } else {
                                    revealAdjacentTiles(tile.r, tile.c);
                                }
                            }
                        }
                        // Right click
                        else if (e.getButton() == MouseEvent.BUTTON3) {
                            if (tile.getText().equals("") && tile.isEnabled()) {
                                tile.setText("🚩");
                            } else if (tile.getText().equals("🚩")) {
                                tile.setText("");
                            }
                        }
                    }
                });

                boardPanel.add(tile);
            }
        }

        frame.setVisible(true);
        setMines();
    }

    public void revealAdjacentTiles(int r, int c) {
        if (r < 0 || r >= numRows || c < 0 || c >= numCols) {
            return;
        }

        MineTile tile = board[r][c];
        if (!tile.isEnabled()) {
            return;
        }
        tile.setEnabled(false);
        tilesClicked += 1;

        int minesFound = countAdjacentMines(r, c);
        tile.setAdjacentMines(minesFound);  // Set the number of adjacent mines

        // Repaint tile to show number with color
        tile.repaint();

        if (minesFound == 0) {
            // Reveal adjacent tiles if there are no adjacent mines
            revealAdjacentTiles(r - 1, c - 1);
            revealAdjacentTiles(r - 1, c);
            revealAdjacentTiles(r - 1, c + 1);
            revealAdjacentTiles(r, c - 1);
            revealAdjacentTiles(r, c + 1);
            revealAdjacentTiles(r + 1, c - 1);
            revealAdjacentTiles(r + 1, c);
            revealAdjacentTiles(r + 1, c + 1);
        }

        if (tilesClicked == numRows * numCols - mineList.size()) {
            gameOver = true;
            textLabel.setText("Mines Cleared!");

            // Show the "Play Again?" dialog
            SwingUtilities.invokeLater(() -> showPlayAgainDialog("You Win!"));
        }
    }

    private void resetGame(boolean returnToWelcome) {
        // Dispose of the current game frame
        frame.dispose();

        if (returnToWelcome) {
            // Return to the welcome screen
            showWelcomeScreen();
        } else {
            // Start a new game
            new Minesweeper();
        }
    }
    private void showPlayAgainDialog(String message) {
        int result = JOptionPane.showOptionDialog(
                frame,
                message + "\nDo you want to play again?",
                "Play Again?",
                JOptionPane.YES_NO_OPTION,
                JOptionPane.QUESTION_MESSAGE,
                null,
                new Object[]{"Yes!", "No!"},
                "Yes!"
        );

        if (result == JOptionPane.YES_OPTION) {
            resetGame(false); // Start a new game
        } else if (result == JOptionPane.NO_OPTION) {
            frame.dispose(); // Close the game frame
        }
    }

    public void setMines() {
        mineList = new ArrayList<>();
        while (mineList.size() < mineCount) {
            int r = random.nextInt(numRows);
            int c = random.nextInt(numCols);
            MineTile tile = board[r][c];
            if (!mineList.contains(tile)) {
                mineList.add(tile);
                tile.setMine(true);
            }
        }
    }

    public int countAdjacentMines(int r, int c) {
        int mineCount = 0;
        for (int dr = -1; dr <= 1; dr++) {
            for (int dc = -1; dc <= 1; dc++) {
                if (dr == 0 && dc == 0) {
                    continue;
                }
                int nr = r + dr;
                int nc = c + dc;
                if (nr >= 0 && nr < numRows && nc >= 0 && nc < numCols) {
                    if (board[nr][nc].isMine()) {
                        mineCount++;
                    }
                }
            }
        }
        return mineCount;
    }

    private void revealMines() {
        for (MineTile tile : mineList) {
            tile.setText("💣");
            tile.setFont(new Font("Arial Unicode MS", Font.BOLD, 36)); // Increase the font size for the mine
            tile.setForeground(Color.BLACK); // Set bomb color to black
        }

        gameOver = true;
        textLabel.setText("Game Over!");

        // Show the "Play Again?" dialog
        SwingUtilities.invokeLater(() -> showPlayAgainDialog("Game Over!"));
    }

    public static void main(String[] args) {
        new Minesweeper();
    }
}


MINESWEEPER.TEST CLASS

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

public class MinesweeperTest {
    private Minesweeper minesweeper;

    @BeforeEach
    public void setUp() {
        minesweeper = new Minesweeper();
        minesweeper.numRows = 5; // Set a smaller board for testing
        minesweeper.numCols = 5;
        minesweeper.mineCount = 5;
        minesweeper.board = new Minesweeper.MineTile[minesweeper.numRows][minesweeper.numCols];

        // Initialize MineTile objects
        for (int r = 0; r < minesweeper.numRows; r++) {
            for (int c = 0; c < minesweeper.numCols; c++) {
                minesweeper.board[r][c] = minesweeper.new MineTile(r, c);
            }
        }

        minesweeper.setMines(); // Set mines for the test
    }

    @Test
    public void testMinePlacement() {
        int mineCount = 0;
        for (int r = 0; r < minesweeper.numRows; r++) {
            for (int c = 0; c < minesweeper.numCols; c++) {
                if (minesweeper.board[r][c].isMine()) {
                    mineCount++;
                }
            }
        }
        assertEquals(minesweeper.mineCount, mineCount, "The number of mines placed should match the mine count.");
    }

    @Test
    public void testCountAdjacentMines() {
        // Manually set mines for this test
        minesweeper.board[1][1].setMine(true);
        minesweeper.board[1][2].setMine(true);
        minesweeper.board[2][1].setMine(true);

        // Count adjacent mines for (2, 2), should be 3
        int adjacentMines = minesweeper.countAdjacentMines(2, 2);
        assertEquals(3, adjacentMines, "There should be 3 adjacent mines.");

        // Count adjacent mines for (0, 0), should be 0
        adjacentMines = minesweeper.countAdjacentMines(0, 0);
        assertEquals(0, adjacentMines, "There should be 0 adjacent mines.");
    }

    @Test
    public void testRevealAdjacentTiles() {
        minesweeper.board[0][0].setMine(true);
        minesweeper.board[0][1].setMine(true);
        minesweeper.board[1][0].setMine(true);
        minesweeper.revealAdjacentTiles(1, 1); // Clicking on a tile adjacent to mines

        assertFalse(minesweeper.board[1][1].isEnabled(), "The tile (1, 1) should be revealed.");
        assertEquals(3, minesweeper.board[1][1].getAdjacentMines(), "The adjacent mines count should be 3.");
    }
}


