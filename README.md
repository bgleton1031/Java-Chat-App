# Java-Chat-App

👨‍💻 This is the code I used for the Secured Messaging App Server:

package SecuredChatServer;

import java.io.*;
import java.net.*;
import java.util.*;

public class SecuredChatServer {
    private static Map<String, PrintWriter> clientWriters = new HashMap<>();

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
        private String username;

        public ClientHandler(Socket socket) {
            this.socket = socket;
        }

        public void run() {
            try {
                in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                out = new PrintWriter(socket.getOutputStream(),true);

                // Get username
                out.println("Enter your username:");
                username = in.readLine();

                //Add this client's writer to the map
                synchronized (clientWriters) {
                    clientWriters.put(username, out);
                }
                // Notify all clients about the new user
                broadcast(username + " has joined the chat.");

                String message;
                while ((message = in.readLine()) != null) {
                    if (message.startsWith ("@")) {
                        int split = message.indexOf(':');
                        if (split > 0) {
                            String targetUsername = message.substring(1, split);
                            String privateMessage = message.substring(split + 1);
                            sendPrivateMessage(targetUsername, privateMessage);
                        }
                    } else {
                        broadcast(username + ": " + message);
                    }
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    socket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                synchronized (clientWriters) {
                    clientWriters.remove(username);
                }
                broadcast(username + " has left the chat.");
            }
        }

        private void broadcast(String message) {
            synchronized (clientWriters) {
                for (PrintWriter writer : clientWriters.values()) {
                    writer.println(message);
                }
            }
        }

        private void sendPrivateMessage(String targetUsername, String message) {
            synchronized (clientWriters) {
                PrintWriter targetWriter = clientWriters.get(targetUsername);
                if (targetWriter != null) {
                    targetWriter.println("Private from " + username + ": " + message);
                    out.println("Private to " + targetUsername + ": " + message);
                } else {
                    out.println("User " + targetUsername + " not found.");
                }
            }
        }
    }
}
// End of Secured Chat Server Code


👨‍💻 This is the code I used for the Secured Messaging App Client:

package SecuredChatClient;

import Client.ChatClient;

import java.io.*;
import java.net.*;
import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class SecuredChatClient {
    private BufferedReader in;
    private PrintWriter out;
    private JFrame frame = new JFrame("Secured Chat Application");
    private JTextField textField = new JTextField(40);
    private JTextArea messageArea = new JTextArea(8,40);

    public SecuredChatClient() {
        //Layout GUI
        textField.setEditable(false);
        messageArea.setEditable(false);
        frame.getContentPane().add(textField, BorderLayout.SOUTH);
        frame.getContentPane().add(new JScrollPane(messageArea), BorderLayout.CENTER);
        frame.pack();

        // Add Listeners
        textField.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e) {
                out.println(textField.getText());
                textField.setText("");
            }
        });
    }

    private void run() throws IOException {
        // Make connection and initialize streams
        Socket socket = new Socket("localhost", 12345);
        in = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        out = new PrintWriter(socket.getOutputStream(), true);

        // Process all messages from server
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
//End of SecuredChatClient code
