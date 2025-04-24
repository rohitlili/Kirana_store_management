# Kirana_store_management
/*A Kirana Store Management System built using Java and JDBC, designed to manage product records such as item details, stock levels, pricing, and daily sales. The application connects to a MySQL database using JDBC for storing and retrieving inventory data, ensuring efficient and reliable store operations.*/
package myPackage;
import java.sql.*;
import java.util.*;

public class KiranaStore1 {
	private static double totalSales = 0;
	private static Connection con;

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);
		try {
			Class.forName("com.mysql.cj.jdbc.Driver");
			System.out.println("Driver registered successfully");
			con = DriverManager.getConnection("jdbc:mysql://localhost:3306/kirana_db", "root", "Rohit@123");
			System.out.println("Connection created");
			int choice;
			while (true) {
				System.out.println();
				System.out.println(" Kirana Store Management ");
				System.out.println("1. Insert Item");
				System.out.println("2. View Current Stock");
				System.out.println("3. Record Sale");
				System.out.println("4. Update Stock");
				System.out.println("5. Delete Item");
				System.out.println("6. Generate Daily Report");
				System.out.println("7. Exit");
				System.out.print("Enter your choice: ");
				choice = sc.nextInt();
				switch (choice) {
				case 1:
					insertItem(sc);
					break;
				case 2:
					viewStock();
					break;
				case 3:
					recordSale(sc);
					break;
				case 4:
					updateStock(sc);
					break;
				case 5:
					deleteItem(sc);
					break;
				case 6:
					generateReport();
					break;
				case 7:
					System.out.println("Exiting the system. Thank you ");
					con.close();
					return;
				default:
					System.out.println("Invalid choice. Please try again.");
				}
			}
		} catch (ClassNotFoundException | SQLException e) {
			e.printStackTrace();
		}
	}

	// Insert Item
	private static void insertItem(Scanner sc) throws SQLException {
		System.out.print("Enter Item ID: ");
		int id = sc.nextInt();
		sc.nextLine();
		System.out.print("Enter Item Name: ");
		String name = sc.nextLine();
		System.out.print("Enter Item Price: ");
		double price = sc.nextDouble();
		System.out.print("Enter Initial Stock: ");
		int stock = sc.nextInt();

		String insertQuery = "INSERT INTO items (id, name, price, stock) VALUES (?, ?, ?, ?)";
		PreparedStatement pstmt = con.prepareStatement(insertQuery);
		pstmt.setInt(1, id);
		pstmt.setString(2, name);
		pstmt.setDouble(3, price);
		pstmt.setInt(4, stock);

		int rowsInserted = pstmt.executeUpdate();
		if (rowsInserted > 0) {
			System.out.println("Item added successfully");
		}
	}

	// View Current Stock
	private static void viewStock() throws SQLException {
		String selectQuery = "SELECT * FROM items";
		PreparedStatement pstmt = con.prepareStatement(selectQuery);
		ResultSet rs = pstmt.executeQuery();
		
		boolean hasItems = false;
		System.out.println("\nCurrent Stock:");
		while (rs.next()) {
			hasItems = true;
			int id = rs.getInt("id");
			String name = rs.getString("name");
			double price = rs.getDouble("price");
			int stock = rs.getInt("stock");
			System.out.println("ID: " + id + ", Name: " + name + ", Price: " + price + ", Stock: " + stock);
		}
		if (!hasItems) {
			System.out.println("No items in stock.");
		}
	}

	// Record Sale
	private static void recordSale(Scanner sc) throws SQLException {
		System.out.print("Enter Item ID to sell: ");
		int id = sc.nextInt();
		System.out.print("Enter Quantity to sell: ");
		int quantity = sc.nextInt();

		String selectQuery = "SELECT price, stock FROM items WHERE id = ?";
		PreparedStatement pstmt = con.prepareStatement(selectQuery);
		pstmt.setInt(1, id);
		ResultSet rs = pstmt.executeQuery();

		if (rs.next()) {
			double price = rs.getDouble("price");
			int stock = rs.getInt("stock");

			if (stock >= quantity) {
				String updateQuery = "UPDATE items SET stock = stock - ? WHERE id = ?";
				PreparedStatement updateStmt = con.prepareStatement(updateQuery);
				updateStmt.setInt(1, quantity);
				updateStmt.setInt(2, id);
				updateStmt.executeUpdate();
				double saleAmount = quantity * price;
				totalSales += saleAmount;
				System.out.println("Sale recorded Amount: " + saleAmount);
			} else {
				System.out.println("scanty stock");
			}
		} else {
			System.out.println("Item not found!");
		}
	}

	// Update Stock
	private static void updateStock(Scanner sc) throws SQLException {
		System.out.print("Enter Item ID to update stock: ");
		int id = sc.nextInt();
		System.out.print("Enter Quantity to add: ");
		int quantity = sc.nextInt();
		String updateQuery = "UPDATE items SET stock = stock + ? WHERE id = ?";
		PreparedStatement pstmt = con.prepareStatement(updateQuery);
		pstmt.setInt(1, quantity);
		pstmt.setInt(2, id);
		int rowsUpdated = pstmt.executeUpdate();
		if (rowsUpdated > 0) {
			System.out.println("Stock updated successfully");
		} else {
			System.out.println("Item not found!");
		}
	}
	// Delete Item
	private static void deleteItem(Scanner sc) throws SQLException {
		System.out.print("Enter Item ID to delete: ");
		int id = sc.nextInt();
		String deleteQuery = "DELETE FROM items WHERE id = ?";
		PreparedStatement pstmt = con.prepareStatement(deleteQuery);
		pstmt.setInt(1, id);
		int rowsDeleted = pstmt.executeUpdate();
		if (rowsDeleted > 0) {
			System.out.println("Item deleted successfully");
		} else {
			System.out.println("Item not found!");
		}
	}

	// Generate Daily Report
	private static void generateReport() {
		System.out.println();
		System.out.println("Daily Report");
		System.out.println("Total Sales: " + totalSales);
	}
}
