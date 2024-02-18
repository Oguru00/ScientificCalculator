
import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.util.Stack;

public class ScientificCalculator extends JFrame implements ActionListener {

    private JTextField inputField;
    private double storedValue = 0;
    private boolean radiansMode = false;
    private StringBuilder currentInput = new StringBuilder();

    public ScientificCalculator() {
        setTitle("Scientific Calculator");
        setSize(1200, 600);
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout());

        getContentPane().setBackground(Color.darkGray);

        inputField = new JTextField();
        inputField.setEditable(false);
        inputField.setFont(new Font("Arial", Font.PLAIN, 20));
        inputField.setHorizontalAlignment(JTextField.RIGHT);
        add(inputField, BorderLayout.NORTH);

        JPanel buttonPanel = new JPanel(new GridLayout(5, 10, 1, 1));
        String[] buttonLabels = {
                "(", ")", "mc", "m+", "m-", "mr", "c", "+/-", "%", "/",
                "x^y", "x^2", "x^3", "sqrt", "exp", "10^x", "7", "8", "9", "*",
                "1/x", "2√x", "3√x", "y√x", "ln", "log", "4", "5", "6", "-",
                "x!", "sin", "cos", "tan", "e", "EE", "1", "2", "3", "+",
                "Rad", "sinh", "cosh", "tanh", "π", "RAND", "0", "del", ".", "="
        };

        for (int i = 0; i < buttonLabels.length; i++) {
            JButton button = new JButton(buttonLabels[i]);
            button.setFont(new Font("Arial", Font.PLAIN, 20));

            if (i % 10 == 9 && (i < 70 || i > 79)) {
                button.setBackground(new Color(255, 165, 0));  // Orange color for last column, excluding specified buttons
                button.setForeground(Color.white);
            } else {
                button.setBackground(Color.darkGray);
                button.setForeground(Color.white);
            }

            button.addActionListener(this);
            buttonPanel.add(button);
        }

        buttonPanel.setBackground(Color.darkGray);
        add(buttonPanel, BorderLayout.CENTER);
    }

    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();

        if (command.equals("=")) {
            evaluateExpression();
        } else {
            switch (command) {
                case "del":
                    handleDelete();
                    break;
                case "Rad":
                    toggleRadiansMode();
                    break;
                case "mc":
                    clearMemory();
                    break;
                case "m+":
                    addToMemory();
                    break;
                case "m-":
                    subtractFromMemory();
                    break;
                case "mr":
                    recallMemory();
                    break;
                default:
                    handleInput(command);
            }
        }
    }

    private void evaluateExpression() {
        try {
            System.out.println(currentInput);
            double result = evaluate(currentInput.toString());
            if (Double.isFinite(result)) {
                storedValue = result;
                inputField.setText(String.valueOf(result));
                currentInput.setLength(0);
                currentInput.append(result);
            } else {
                inputField.setText("Error");
                currentInput.setLength(0);
            }
        } catch (Exception e) {
            inputField.setText("Error");
            currentInput.setLength(0);
        }
    }

    private double evaluate(String expression) {
        expression = expression.replaceAll("\\s+", ""); // remove all spaces from the expression
        String[] tokens = expression.split("(?<=\\d)(?=\\D)|(?<=\\D)(?=\\d)"); // split expression based on digits and non-digits
        Stack<Double> values = new Stack<>();
        Stack<String> operators = new Stack<>();

        for (String token : tokens) {
            if (token.isEmpty()) {
                continue;
            }

            if (isNumber(token)) {
                values.push(Double.parseDouble(token));
            } else if (isOperator(token)) {
                if (values.isEmpty()) {
                    throw new IllegalArgumentException("Missing operand before operator");
                }

                while (!operators.isEmpty() && precedence(token) <= precedence(operators.peek())) {
                    double val2 = values.pop();
                    double val1 = values.pop();
                    String op = operators.pop();
                    values.push(applyOperator(op, val1, val2));
                }
                operators.push(token);
            } else {
                throw new IllegalArgumentException("Invalid token: " + token);
            }
        }

        if (operators.size() > values.size()) {
            throw new IllegalArgumentException("Missing operand after operator");
        }

        while (!operators.isEmpty()) {
            double val2 = values.pop();
            double val1 = values.pop();
            String op = operators.pop();
            values.push(applyOperator(op, val1, val2));
        }

        return values.pop();
    }


    private boolean isNumber(String token) {
        try {
            Double.parseDouble(token);
            return true;
        } catch (NumberFormatException e) {
            return false;
        }
    }

    private boolean isOperator(String token) {
        return token.matches("[+\\-*/^]");
    }

    private int precedence(String operator) {
        switch (operator) {
            case "+":
            case "-":
                return 1;
            case "*":
            case "/":
                return 2;
            case "^":
                return 3;
            default:
                return 0;
        }
    }

    private double applyOperator(String operator, double a, double b) {
        switch (operator) {
            case "+":
                return a + b;
            case "-":
                return a - b;
            case "*":
                return a * b;
            case "/":
                if (b != 0) {
                    return a / b;
                } else {
                    throw new ArithmeticException("Division by zero");
                }
            case "^":
                return Math.pow(a, b);
            default:
                throw new IllegalArgumentException("Invalid operator: " + operator);
        }
    }

    private void handleDelete() {
        if (currentInput.length() > 0) {
            currentInput.deleteCharAt(currentInput.length() - 1);
            inputField.setText(currentInput.toString());
        }
    }

    private void toggleRadiansMode() {
        radiansMode = !radiansMode;
    }

    private void clearMemory() {
        storedValue = 0;
    }

    private void addToMemory() {
        storedValue += Double.parseDouble(currentInput.toString());
    }

    private void subtractFromMemory() {
        storedValue -= Double.parseDouble(currentInput.toString());
    }

    private void recallMemory() {
        currentInput.setLength(0);
        currentInput.append(storedValue);
        inputField.setText(currentInput.toString());
    }

    private void handleInput(String input) {
        currentInput.append(input);
        inputField.setText(currentInput.toString());
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            ScientificCalculator calculator = new ScientificCalculator();
            calculator.setVisible(true);
        });
    }
}
