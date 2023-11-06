# battleship
help me with my battleship game
package battleship;
import java.util.Scanner;

class Game {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        Game game = new Game(scanner);
        game.play();
        scanner.close();
    }

    private final Scanner scanner;
    private final Board board;

    public Game(Scanner scanner) {
        this.scanner = scanner;
        this.board = new Board();
    }

    public void play() {
        String[] shipNames = {"Aircraft Carrier", "Battleship", "Submarine", "Cruiser", "Destroyer"};
        int[] shipSizes = {5, 4, 3, 3, 2};

        board.displayGameBoard();

        for (int i = 0; i < 5; i++) {
            boolean placementSuccessful = false;
            while (!placementSuccessful) {
                System.out.println("Enter the coordinates of the " + shipNames[i] + " (" + shipSizes[i] + " cells):");
                String input = scanner.nextLine();

                if (board.placeShip(shipNames[i], shipSizes[i], input)) {
                    placementSuccessful = true;
                    board.displayGameBoard(false);
                }
            }
        }
        System.out.println("The game starts!");

        int totalShips = 17; // Total number of ships in the game
        int sunkShips = 0; // Number of sunk ships

        while (sunkShips < totalShips) {
            board.displayGameBoard();
            System.out.println("Take a shot!");

            boolean shootingSuccessful = false;
            while (!shootingSuccessful) {
                String input = scanner.nextLine();
                Coordinate shotCoordinate = Coordinate.fromString(input);

                if (Coordinate.isValid(shotCoordinate)) {
                    if (board.isHit(shotCoordinate)) {
                        board.displayGameBoard();
                        System.out.println("You hit a ship!");
                        System.out.println();
                        board.displayGameBoard(false);
                        shootingSuccessful = true;

                        // Check if a ship is sunk
                        if (board.isShipSunk(shotCoordinate)) {
                            System.out.println("You sank a ship!");
                            sunkShips++;

                            // Check if all ships are sunk
                            if (sunkShips == totalShips) {
                                System.out.println("You sank the last ship. You won. Congratulations!");
                                return; // End the game
                            }
                        }
                    } else {
                        board.displayGameBoard();
                        System.out.println("You missed!");
                    }
                } else {
                    System.out.println("Invalid input. Please try again.");
                }
            }
        }
    }
}
package battleship;
import java.util.Scanner;

class Board {
    private static final int ROWS = 10;
    private static final int COLUMNS = 10;
    private char[][] board = new char[ROWS][COLUMNS];

    public Board() {
        for (int i = 0; i < ROWS; i++) {
            for (int j = 0; j < COLUMNS; j++) {
                board[i][j] = '~';
            }
        }
    }

    public void displayGameBoard() {
        displayGameBoard(true);
    }

    public void displayGameBoard(boolean hideShips) {
        System.out.println("  1 2 3 4 5 6 7 8 9 10");
        for (int row = 0; row < ROWS; row++) {
            System.out.print((char) ('A' + row));
            for (int col = 0; col < COLUMNS; col++) {
                char cell = board[row][col];
                if (hideShips && cell == 'O') {
                    System.out.print(" " + "~");
                } else {
                    System.out.print(" " + cell);
                }
            }
            System.out.println();
        }
    }

    public boolean placeShip(String shipName, int shipLength, String coordinatesInput) {
        String[] coordinates = coordinatesInput.split(" ");
        try {
            Coordinate startCoord = Coordinate.fromString(coordinates[0]);
            Coordinate endCoord = Coordinate.fromString(coordinates[1]);

            if (!Coordinate.isValid(startCoord) || !Coordinate.isValid(endCoord)) {
                throw new Exception("Invalid coordinates.");
            }

            Ship ship = new Ship(shipName, shipLength, startCoord, endCoord);

            if (!isShipPlacementValid(ship)) {
                throw new Exception("Invalid ship placement. Check ship size or overlapping.");
            }

            placeOnBoard(ship);
            return true;
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
        return false;
    }

    public void placeOnBoard(Ship ship) {
        int startRow = ship.getStartCoord().getRow();
        int startCol = ship.getStartCoord().getColumn();
        int endRow = ship.getEndCoord().getRow();
        int endCol = ship.getEndCoord().getColumn();

        for (int row = startRow; row <= endRow; row++) {
            for (int col = startCol; col <= endCol; col++) {
                board[row][col] = 'O';
            }
        }
    }

    public boolean isShipPlacementValid(Ship ship) {
        int startRow = ship.getStartCoord().getRow();
        int startCol = ship.getStartCoord().getColumn();
        int endRow = ship.getEndCoord().getRow();
        int endCol = ship.getEndCoord().getColumn();

        int shipLength = ship.getLength();

        if (startRow != endRow && startCol != endCol) {
            return false; // Not a valid placement (diagonal)
        }

        if (hasConflictingRestrictedCells(ship)) {
            return false; // There is a neighboring ship in restricted cells
        }

        return shipLength == (endRow - startRow) + (endCol - startCol) + 1;
    }

    public boolean hasConflictingRestrictedCells(Ship ship) {
        int startRow = ship.getStartCoord().getRow();
        int startCol = ship.getStartCoord().getColumn();
        int endRow = ship.getEndCoord().getRow();
        int endCol = ship.getEndCoord().getColumn();

        for (int row = startRow - 1; row <= endRow + 1; row++) {
            for (int col = startCol - 1; col <= endCol + 1; col++) {
                if (row >= 0 && row < ROWS && col >= 0 && col < COLUMNS) {
                    if (board[row][col] != '~') {
                        return true;
                    }
                }
            }
        }

        return false;
    }

    public boolean isHit(Coordinate coordinate) {
        int row = coordinate.getRow();
        int col = coordinate.getColumn();

        if (board[row][col] == 'O') {
            board[row][col] = 'X';

               return true;

        } else {
            board[row][col] = 'M';
            return false;
        }
    }
    public boolean areAllShipsSunk() {
        for (int row = 0; row < ROWS; row++) {
            for (int col = 0; col < COLUMNS; col++) {
                if (board[row][col] == 'O') {
                    return false; // There's still at least one ship left
                }
            }
        }
        return true; // All ships have been sunk
    }

    public boolean isShipAt(Coordinate coordinate) {
        return
                board[coordinate.getRow()][coordinate.getColumn()] == 'O'
                        || board[coordinate.getRow()][coordinate.getRow()] == 'X';
    }
    public boolean isShipSunk(Coordinate coordinate) {
        int row = coordinate.getRow();
        int col = coordinate.getColumn();

        char shipSymbol = board[row][col];
        if (shipSymbol == 'X') {
            // This is part of a ship, check if the ship is sunk

            // Determine the boundaries of the ship
            int startRow = row;
            int startCol = col;
            int endRow = row;
            int endCol = col;

            for (int i = row; i >= 0; i--) {
                if (!isShipAt(new Coordinate(i, col))) {
                    break;
                }
                startRow = i;
            }

            for (int i = col; i >= 0; i--) {
                if (!isShipAt(new Coordinate(row, i))) {
                    break;
                }
                startCol = i;
            }

            for (int i = row; i < ROWS; i++) {
                if (!isShipAt(new Coordinate(i, col))) {
                    break;
                }
                endRow = i;
            }

            for (int i = col; i < COLUMNS; i++) {
                if (!isShipAt(new Coordinate(row, i))) {
                    break;
                }
                endCol = i;
            }

            // Check if the ship is completely sunk
            for (int i = startRow; i <= endRow; i++) {
                for (int j = startCol; j <= endCol; j++) {
                    if (board[i][j] == 'O') {
                        return false; // The ship is not completely sunk
                    }
                }
            }

            return true; // The ship is completely sunk
        }

        return false; // This is not a part of a ship
    }
}
package battleship;

class Ship {
    private String name;
    private int length;
    private Coordinate startCoord;
    private Coordinate endCoord;

    public Ship(String name, int length, Coordinate startCoord, Coordinate endCoord) {
        this.name = name;
        this.length = length;
        if (startCoord.getRow() > endCoord.getRow() || startCoord.getColumn() > endCoord.getColumn()) {
            this.startCoord = endCoord;
            this.endCoord = startCoord;
        } else {
            this.startCoord = startCoord;
            this.endCoord = endCoord;
        }
    }

    public Coordinate getStartCoord() {
        return startCoord;
    }

    public Coordinate getEndCoord() {
        return endCoord;
    }

    public int getLength() {
        return length;
    }
}
package battleship;
public class Coordinate {
    private int row;
    private int column;

    public Coordinate(int row, int column) {
        this.row = row;
        this.column = column;
    }

    public static Coordinate fromString(String coordinateString) {
        int row = coordinateString.charAt(0) - 'A';
        int column = Integer.parseInt(coordinateString.substring(1)) - 1;
        return new Coordinate(row, column);
    }

    public static boolean isValid(Coordinate coordinate) {
        return (coordinate != null) && (coordinate.row >= 0 && coordinate.row < 10) && (coordinate.column >= 0 && coordinate.column < 10);
    }

    public int getRow() {
        return row;
    }

    public int getColumn() {
        return column;
    }
}
Execution failed

java.lang.AssertionError: Wrong answer in test #1

At the end of the game, your program should print a congratulatory message to the winner: You sank the last ship. You won. Congratulations!

Please find below the output of your program during this failed test.
Note that the '>' character indicates the beginning of the input line.

---

[last 250 lines of output are shown, 596 skipped]
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
H ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
I ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
J ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
You missed!
> G8
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ ~ ~ ~ M ~ ~
H ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
I ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
J ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
You missed!
> G5
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
I ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
J ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
You missed!
> H2
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
J ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
You missed!
> I2
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
You hit a ship!

  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ O ~ ~ ~ ~ ~ O O O
You sank a ship!
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ ~ ~ ~ ~ ~ ~ ~ ~ ~
Take a shot!
> J2
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ ~ ~ ~ ~ ~
You hit a ship!

  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ ~ ~ O O O
You sank a ship!
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ ~ ~ ~ ~ ~
Take a shot!
> J6
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ ~ ~ ~
You missed!
> J8
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ X ~ ~
You hit a ship!

  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ X O O
You sank a ship!
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ X ~ ~
Take a shot!
> J9
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ X X ~
You hit a ship!

  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ X X O
You sank a ship!
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ X X ~
Take a shot!
> J10
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ X X X
You hit a ship!

  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ X X X
You sank a ship!
  1 2 3 4 5 6 7 8 9 10
A X M ~ ~ ~ ~ ~ ~ ~ ~
B X ~ ~ ~ ~ ~ ~ ~ X ~
C X ~ ~ ~ ~ ~ ~ ~ X ~
D X ~ ~ ~ ~ ~ ~ ~ X ~
E ~ ~ ~ M ~ ~ ~ ~ M ~
F ~ ~ X X X X X ~ ~ M
G ~ ~ ~ ~ M ~ ~ M ~ ~
H ~ M ~ ~ ~ ~ ~ ~ ~ ~
I ~ X ~ ~ ~ ~ ~ ~ ~ ~
J ~ X ~ ~ ~ M ~ X X X
Take a shot!
	at org.junit.Assert.fail(Assert.java:89)
	at org.hyperskill.hstest.stage.StageTest.start(StageTest.java:203)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:568)
	at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:59)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:56)
	at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
	at org.junit.runners.ParentRunner$3.evaluate(ParentRunner.java:306)
	at org.junit.runners.BlockJUnit4ClassRunner$1.evaluate(BlockJUnit4ClassRunner.java:100)
	at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:366)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:103)
	at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:63)
	at org.junit.runners.ParentRunner$4.run(ParentRunner.java:331)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:79)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:329)
	at org.junit.runners.ParentRunner.access$100(ParentRunner.java:66)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:293)
	at org.junit.runners.ParentRunner$3.evaluate(ParentRunner.java:306)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:413)
	at org.gradle.api.internal.tasks.testing.junit.JUnitTestClassExecutor.runTestClass(JUnitTestClassExecutor.java:110)
	at org.gradle.api.internal.tasks.testing.junit.JUnitTestClassExecutor.execute(JUnitTestClassExecutor.java:58)
	at org.gradle.api.internal.tasks.testing.junit.JUnitTestClassExecutor.execute(JUnitTestClassExecutor.java:38)
	at org.gradle.api.internal.tasks.testing.junit.AbstractJUnitTestClassProcessor.processTestClass(AbstractJUnitTestClassProcessor.java:62)
	at org.gradle.api.internal.tasks.testing.SuiteTestClassProcessor.processTestClass(SuiteTestClassProcessor.java:51)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:77)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:568)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:36)
	at org.gradle.internal.dispatch.ReflectionDispatch.dispatch(ReflectionDispatch.java:24)
	at org.gradle.internal.dispatch.ContextClassLoaderDispatch.dispatch(ContextClassLoaderDispatch.java:33)
	at org.gradle.internal.dispatch.ProxyDispatchAdapter$DispatchingInvocationHandler.invoke(ProxyDispatchAdapter.java:94)
	at jdk.proxy1/jdk.proxy1.$Proxy2.processTestClass(Unknown Source)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker$2.run(TestWorker.java:176)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker.executeAndMaintainThreadName(TestWorker.java:129)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker.execute(TestWorker.java:100)
	at org.gradle.api.internal.tasks.testing.worker.TestWorker.execute(TestWorker.java:60)
	at org.gradle.process.internal.worker.child.ActionExecutionWorker.execute(ActionExecutionWorker.java:56)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:133)
	at org.gradle.process.internal.worker.child.SystemApplicationClassLoaderWorker.call(SystemApplicationClassLoaderWorker.java:71)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.run(GradleWorkerMain.java:69)
	at worker.org.gradle.process.internal.worker.GradleWorkerMain.main(GradleWorkerMain.java:74)
