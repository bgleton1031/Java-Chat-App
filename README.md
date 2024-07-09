# Java-Chat-App

This is the code I've used for the **unsecured** messaging server.


If you're looking for the code for the **secured** version, click on this Do you want the SECURED CODE? You can click on this [REPOSITORY](https://github.com/bgleton1031/Java-Chat-App/blob/Secured-Messaging-App-Code/README.md). 

package Server;

import java.io.*;
import java.net.*;
import java.util.*;

public class ChatServer {
    private static Set<PrintWriter> clientWriters = new HashSet<>();

    public static void main(String[] args) throws IOException {
        System.out.println("Chat server started...");
        ServerSocket serverSocket = new ServerSocket(12345);

        while (true) {
            new ClientHandler(serverSocket.accept()).start();
        }
    }

    private static class ClientHandler extends Thread {
        private Socket socket;
        private PrintWriter out;
        private BufferedReader in;

        public ClientHandler(Socket socket) {
            this.socket = socket;
        }

        public void run() {
            try {
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                out = new PrintWriter(socket.getOutputStream(), true);
                synchronized (clientWriters) {
                    clientWriters.add(out);
                }
                String message;
                while ((message = in.readLine()) != null) {
                    System.out.println("Received: " + message);
                    synchronized (clientWriters) {
                        for (PrintWriter writer : clientWriters) {
                            writer.println(message);
                        }
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    socket.close();
                }catch (IOException e) {
                    e.printStackTrace();
                }
                synchronized (clientWriters) {
                    clientWriters.remove(out);
                }
            }
        }
    }
}
//End of ChatServer Code



This is the code for the **unsecured** messaging app.

package Client;

import java.io.*;
import java.net.*;
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class ChatClient {
    private BufferedReader in;
    private PrintWriter out;
    public JFrame frame = new JFrame("Chat Application");
    private JTextField textField = new JTextField(40);
    private JTextArea messageArea = new JTextArea(8,40);

    public ChatClient() {
        // Layout GUI (Graphic User Interface)
        textField.setEditable(false);
        messageArea.setEditable(false);
        frame.getContentPane().add(textField, BorderLayout.SOUTH);
        frame.getContentPane().add(new JScrollPane(messageArea), BorderLayout.CENTER);
        frame.pack();

        // Add Listeners
        textField.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                out.println(textField.getText());
                textField.setText("");
            }
        });
    }

    public void run() throws IOException {
        //Make connection and initialize streams
        while (true) {
            String line = in.readLine();
            if (line == null) {
                break;
            }
            messageArea.append(line + "\n");
        }
    }
    public static void main(String[] args) throws Exception {
        ChatClient client = new ChatClient();
        client.frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        client.frame.setVisible(true);
        client.run();
    }
}
//End of ChatClient code


Do you want the SECURED CODE? You can click on this c
