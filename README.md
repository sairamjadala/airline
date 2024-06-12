# airline
Airline reservation system

import javax.swing.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.text.DecimalFormat;
import java.util.ArrayList;
import java.util.List;

class Expense {
    private String description;
    private double amount;

    public Expense(String description, double amount) {
        this.description = description;
        this.amount = amount;
    }

    public String getDescription() {
        return description;
    }

    public double getAmount() {
        return amount;
    }

    @Override
    public String toString() {
        return "Description: " + description + ", Amount: $" + amount;
    }
}

public class ExpenseTrackerGUI {
    private List<Expense> expenses = new ArrayList<>();
    private double totalExpenses = 0.0;
    private double budgetLimit = 0.0; // Initialize with 0 initially

    private JFrame frame;
    private JTextArea textArea;
    private JTextField descriptionField;
    private JTextField amountField;
    private JTextField budgetField; // Added field for user-defined budget
    private JLabel budgetLabel;

    private boolean budgetEntered = false; // Flag to check if the budget has been entered

    public ExpenseTrackerGUI() {
        frame = new JFrame("Expense Tracker");
        frame.setSize(400, 300);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

        createGUI();

        frame.setVisible(true);
    }

    private void createGUI() {
        JPanel panel = new JPanel();
        frame.add(panel);
        placeComponents(panel);

        frame.getContentPane().setLayout(new BoxLayout(frame.getContentPane(), BoxLayout.Y_AXIS));
    }

    private void placeComponents(JPanel panel) {
        panel.setLayout(null);

        JLabel titleLabel = new JLabel("Expense Tracker");
        titleLabel.setBounds(10, 20, 150, 25);
        panel.add(titleLabel);

        JLabel budgetPromptLabel = new JLabel("Enter Budget:");
        budgetPromptLabel.setBounds(10, 50, 100, 25);
        panel.add(budgetPromptLabel);

        budgetField = new JTextField();
        budgetField.setBounds(120, 50, 150, 25);
        panel.add(budgetField);

        JButton setBudgetButton = new JButton("Set Budget");
        setBudgetButton.setBounds(280, 50, 100, 25);
        panel.add(setBudgetButton);

        JLabel descriptionLabel = new JLabel("Description:");
        descriptionLabel.setBounds(10, 80, 80, 25);
        panel.add(descriptionLabel);

        descriptionField = new JTextField();
        descriptionField.setBounds(100, 80, 150, 25);
        panel.add(descriptionField);

        JLabel amountLabel = new JLabel("Amount:");
        amountLabel.setBounds(10, 110, 80, 25);
        panel.add(amountLabel);

        amountField = new JTextField();
        amountField.setBounds(100, 110, 150, 25);
        panel.add(amountField);

        JButton addExpenseButton = new JButton("Add Expense");
        addExpenseButton.setBounds(10, 140, 150, 25);
        panel.add(addExpenseButton);

        JButton deleteExpenseButton = new JButton("Delete Expense");
        deleteExpenseButton.setBounds(10, 170, 150, 25);
        panel.add(deleteExpenseButton);

        JButton viewExpensesButton = new JButton("View Expenses");
        viewExpensesButton.setBounds(10, 200, 150, 25);
        panel.add(viewExpensesButton);

        this.budgetLabel = new JLabel("Budget Remaining: $" + formatBudget(budgetLimit - totalExpenses));
        this.budgetLabel.setBounds(180, 20, 200, 25);
        panel.add(this.budgetLabel);

        textArea = new JTextArea();
        JScrollPane scrollPane = new JScrollPane(textArea);
        scrollPane.setBounds(180, 80, 200, 150);
        panel.add(scrollPane);

        setBudgetButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                setBudget();
            }
        });

        addExpenseButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                addExpense();
            }
        });

        deleteExpenseButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                deleteExpense();
            }
        });

        viewExpensesButton.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                viewExpenses();
            }
        });
    }

    private void setBudget() {
        if (budgetEntered) {
            JOptionPane.showMessageDialog(frame, "Budget already set. Cannot change it.");
            return;
        }

        String budgetStr = budgetField.getText();

        if (budgetStr.isEmpty()) {
            JOptionPane.showMessageDialog(frame, "Please enter a budget.");
            return;
        }

        try {
            budgetLimit = Double.parseDouble(budgetStr);

            if (budgetLimit <= 0) {
                JOptionPane.showMessageDialog(frame, "Please enter a valid budget greater than zero.");
                return;
            }

            budgetEntered = true;
            JOptionPane.showMessageDialog(frame, "Budget set successfully.");
        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(frame, "Invalid budget. Please enter a valid number.");
        }
    }

    private void addExpense() {
        if (!budgetEntered) {
            JOptionPane.showMessageDialog(frame, "Please set the budget before adding expenses.");
            return;
        }

        String description = descriptionField.getText();
        String amountStr = amountField.getText();

        if (description.isEmpty() || amountStr.isEmpty()) {
            JOptionPane.showMessageDialog(frame, "Please enter both description and amount.");
            return;
        }

        try {
            double amount = Double.parseDouble(amountStr);

            while (totalExpenses + amount > budgetLimit && !expenses.isEmpty()) {
                // Remove the oldest expense(s) until the budget is within limits
                Expense oldestExpense = expenses.remove(0);
                totalExpenses -= oldestExpense.getAmount();
            }

            if (totalExpenses + amount > budgetLimit) {
                JOptionPane.showMessageDialog(frame, "Exceeds budget limit. Cannot add expense.");
                return;
            }

            expenses.add(new Expense(description, amount));
            totalExpenses += amount;
            updateBudgetLabel();
            JOptionPane.showMessageDialog(frame, "Expense added successfully.");
            descriptionField.setText("");
            amountField.setText("");
        } catch (NumberFormatException ex) {
            JOptionPane.showMessageDialog(frame, "Invalid amount. Please enter a valid number.");
        }
    }

    private void deleteExpense() {
        if (expenses.isEmpty()) {
            JOptionPane.showMessageDialog(frame, "No expenses to delete.");
            return;
        }

        // Remove the oldest expense
        Expense removedExpense = expenses.remove(0);
        totalExpenses -= removedExpense.getAmount();
        updateBudgetLabel();
        JOptionPane.showMessageDialog(frame, "Expense deleted successfully.");
    }

    private void viewExpenses() {
        if (expenses.isEmpty()) {
            textArea.setText("No expenses recorded yet.");
        } else {
            StringBuilder stringBuilder = new StringBuilder();
            stringBuilder.append("List of Expenses:\n");
            for (Expense expense : expenses) {
                stringBuilder.append(expense).append("\n");
            }
            stringBuilder.append("Total Expenses: $").append(totalExpenses);
            textArea.setText(stringBuilder.toString());
        }
    }

    private void updateBudgetLabel() {
        double remainingBudget = budgetLimit - totalExpenses;
        this.budgetLabel.setText("Budget Remaining: $" + formatBudget(remainingBudget));

        if (remainingBudget < 0) {
            this.budgetLabel.setForeground(java.awt.Color.RED);
        } else {
            this.budgetLabel.setForeground(java.awt.Color.BLACK);
        }
    }

    private String formatBudget(double budget) {
        DecimalFormat df = new DecimalFormat("#.##");
        return df.format(budget);
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(new Runnable() {
            public void run() {
                new ExpenseTrackerGUI();
            }
        });
    }
}
