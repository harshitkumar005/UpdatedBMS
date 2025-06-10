import java.sql.*;
import java.util.InputMismatchException;
import java.util.Scanner;

public class BankManagement {

    static final String DB_URL = "jdbc:mysql://localhost:3306/BankDB";
    static final String USER = "root";
    static final String PASS = "your_password"; // Update your password

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        try (Connection conn = DriverManager.getConnection(DB_URL, USER, PASS)) {
            System.out.println("Connected to database successfully.");
            while (true) {
                showMenu();
                int choice = getValidIntInput(scanner, "Enter choice: ");
                switch (choice) {
                    case 1 -> createAccount(conn, scanner);
                    case 2 -> viewAccount(conn, scanner);
                    case 3 -> deposit(conn, scanner);
                    case 4 -> withdraw(conn, scanner);
                    case 5 -> {
                        System.out.println("Thank you for using the Bank Management System!");
                        return;
                    }
                    default -> System.out.println("⚠ Invalid choice. Please try again.");
                }
            }
        } catch (SQLException e) {
            System.err.println("❌ Database connection failed: " + e.getMessage());
        }
    }

    static void showMenu() {
        System.out.println("\n===== Bank Management System =====");
        System.out.println("1. Create Account");
        System.out.println("2. View Account");
        System.out.println("3. Deposit");
        System.out.println("4. Withdraw");
        System.out.println("5. Exit");
    }

    static int getValidIntInput(Scanner scanner, String prompt) {
        while (true) {
            try {
                System.out.print(prompt);
                return scanner.nextInt();
            } catch (InputMismatchException e) {
                System.out.println("⚠ Please enter a valid number.");
                scanner.next(); // clear invalid input
            }
        }
    }

    static double getValidDoubleInput(Scanner scanner, String prompt) {
        while (true) {
            try {
                System.out.print(prompt);
                double input = scanner.nextDouble();
                if (input < 0) {
                    System.out.println("⚠ Amount cannot be negative.");
                } else {
                    return input;
                }
            } catch (InputMismatchException e) {
                System.out.println("⚠ Please enter a valid amount.");
                scanner.next();
            }
        }
    }

    static void createAccount(Connection conn, Scanner scanner) throws SQLException {
        scanner.nextLine(); // clear buffer
        System.out.print("Enter your name: ");
        String name = scanner.nextLine().trim();
        if (name.isEmpty()) {
            System.out.println("⚠ Name cannot be empty.");
            return;
        }

        double balance = getValidDoubleInput(scanner, "Enter initial deposit: ");

        String sql = "INSERT INTO accounts (name, balance) VALUES (?, ?)";
        try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setString(1, name);
            pstmt.setDouble(2, balance);
            pstmt.executeUpdate();
            System.out.println("✅ Account created successfully!");
        }
    }

    static void viewAccount(Connection conn, Scanner scanner) throws SQLException {
        int accNo = getValidIntInput(scanner, "Enter account number: ");

        String sql = "SELECT * FROM accounts WHERE acc_no = ?";
        try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setInt(1, accNo);
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    System.out.println("Account Number: " + rs.getInt("acc_no"));
                    System.out.println("Name: " + rs.getString("name"));
                    System.out.println("Balance: ₹" + rs.getDouble("balance"));
                } else {
                    System.out.println("⚠ Account not found.");
                }
            }
        }
    }

    static void deposit(Connection conn, Scanner scanner) throws SQLException {
        int accNo = getValidIntInput(scanner, "Enter account number: ");
        double amount = getValidDoubleInput(scanner, "Enter amount to deposit: ");

        String sql = "UPDATE accounts SET balance = balance + ? WHERE acc_no = ?";
        try (PreparedStatement pstmt = conn.prepareStatement(sql)) {
            pstmt.setDouble(1, amount);
            pstmt.setInt(2, accNo);
            int rows = pstmt.executeUpdate();
            if (rows > 0) {
                System.out.println("✅ Deposit successful.");
            } else {
                System.out.println("⚠ Account not found.");
            }
        }
    }

    static void withdraw(Connection conn, Scanner scanner) throws SQLException {
        int accNo = getValidIntInput(scanner, "Enter account number: ");
        double amount = getValidDoubleInput(scanner, "Enter amount to withdraw: ");

        String checkSql = "SELECT balance FROM accounts WHERE acc_no = ?";
        try (PreparedStatement checkStmt = conn.prepareStatement(checkSql)) {
            checkStmt.setInt(1, accNo);
            ResultSet rs = checkStmt.executeQuery();
            if (rs.next()) {
                double balance = rs.getDouble("balance");
                if (balance >= amount) {
                    String withdrawSql = "UPDATE accounts SET balance = balance - ? WHERE acc_no = ?";
                    try (PreparedStatement pstmt = conn.prepareStatement(withdrawSql)) {
                        pstmt.setDouble(1, amount);
                        pstmt.setInt(2, accNo);
                        pstmt.executeUpdate();
                        System.out.println("✅ Withdrawal successful.");
                    }
                } else {
                    System.out.println("⚠ Insufficient balance.");
                }
            } else {
                System.out.println("⚠ Account not found.");
            }
        }
    }
}
