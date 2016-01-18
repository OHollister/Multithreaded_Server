# Multithreaded_Server
Multithreaded Server program capable of handling multiple clients.

//CLASS  MultithreadedServer
package Multithreaded;
//12/2/2015
//Olga Hollister

import java.io.*;
import java.net.*;
import java.util.*;
import java.awt.*;

import javax.swing.*;

public class MultithreadedServer extends JFrame {
	// Text area for displaying contents
	private JTextArea jta = new JTextArea();
	Map m1 = new HashMap();

	public static void main(String[] args) {
		new MultithreadedServer();
	}

	public MultithreadedServer() {
		// Place text area on the frame
		setLayout(new BorderLayout());
		add(new JScrollPane(jta), BorderLayout.CENTER);

		setTitle("MultiThreadServer");
		setSize(500, 300);
		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		setVisible(true); // It is necessary to show the frame here!

		try {
			// Create a server socket
			ServerSocket serverSocket = new ServerSocket(8000); // where 8000 is
																// a port number
			jta.append("MultiThreadServer started at " + new Date() + '\n'); // java.util.Date.Date()

			// Number a client
			int clientNo = 1;

			while (true) {
				// Listen for a new connection request
				Socket socket = serverSocket.accept();// Listens for a connection to be made to this socket and accepts it.

				// Display the client number
				jta.append("Starting thread for client " + clientNo + " at "
						+ new Date() + '\n');

				// Find the client's host name, and IP address
				InetAddress inetAddress = socket.getInetAddress(); // java.net.InetAddress
																	// .This
																	// class
																	// represents
																	// an
																	// Internet
																	// Protocol
																	// (IP)
																	// address.

				jta.append("Client " + clientNo + "'s host name is "
						+ inetAddress.getHostName() + "\n"); // java.net.InetAddress.getHostName()
																// Gets the host name for this IP address.

				jta.append("Client " + clientNo + "'s IP Address is "
						+ inetAddress.getHostAddress() + "\n"); // Returns the  IP address string in textual presentation.

				// Create a new thread for the connection
				HandleAClient task = new HandleAClient(socket); // HandleAClient
																// is a inner
																// class

				// Start the new thread
				new Thread(task).start(); // Causes this thread to begin
											// execution; the Java Virtual
											// Machine calls the run method of
											// this thread.

				// Increment clientNo
				clientNo++;
			}
		} catch (IOException ex) {
			System.err.println(ex);
		}
	}

	// Inner class
	// Define the thread class for handling new connection
	class HandleAClient implements Runnable {
		private Socket socket; // A connected socket . A socket is an endpoint
								// for communication between two machines.

		/** Construct a thread */
		public HandleAClient(Socket socket) {
			this.socket = socket;
		}

		/** Run a thread */
		public void run() {
			try {
				// Create data input and output streams
				DataInputStream inputFromClient = new DataInputStream(
						socket.getInputStream());
				DataOutputStream outputToClient = new DataOutputStream(
						socket.getOutputStream());

				// Continuously serve the client
				while (true) {
					// Receive data from the client
					int n = inputFromClient.readInt();
					String name = inputFromClient.readUTF();
					String phone = inputFromClient.readUTF();
					String error = "Entered data does not exist:";

					switch (n) {
					case 1: // add to map

						if (m1.containsValue(name) && m1.containsValue(phone)) {
							outputToClient.writeUTF(name + phone);
						}
						m1.put(name, phone); // add name and phone to the
												// collection
						outputToClient.writeUTF(name); // Send back to the
														// client
						outputToClient.writeUTF(phone);
						jta.append("Name received from client: " + name + '\n');
						jta.append("Phone received from client: " + phone
								+ '\n');

						break;
					case 2:

						String str = (String) m1.get(name);
						if (str == null) {
							jta.append("Entered name does not exist: " + name
									+ '\n');
							outputToClient.writeUTF(error);
							outputToClient.writeUTF(error);
						} else {
							jta.append("Entered name: " + name + '\n');
							String st = (String) m1.get(phone);
							jta.append("Phone found: " + str + '\n');

							outputToClient.writeUTF(name); // Send back to the
															// client
							outputToClient.writeUTF(str);

						}
						break;

					}
					// System.out.println(m1); use to check values

				}
			} catch (IOException e) {
				System.err.println(e);
			}
		}
	}
}
 
 
  //CLASS Client
 package Multithreaded;
//12/2/2015
//Olga Hollister

import java.io.*;
import java.net.*;
import java.awt.*;
import java.awt.event.*;

import javax.swing.*;

public class Client extends JFrame {
  // Text field for receiving radius
  private JTextField jtf = new JTextField();
  private JTextField jtf1 = new JTextField();
  private JTextField jtf2 = new JTextField();
  private JTextField jtf3 = new JTextField();

  // Text area to display contents
  private JTextArea jta = new JTextArea();

  // IO streams
  private DataOutputStream toServer;
  private DataInputStream fromServer;

  public static void main(String[] args) {
    new Client();
  }

  public Client() {
    // Panel p to hold the label and text field
    JPanel p = new JPanel();
    p.setLayout(new GridLayout(7,1));
    
    p.add(new JLabel("1 send name & phone; 2 request for Phone#"), BorderLayout.WEST);
    
    p.add(new JLabel("Command"), BorderLayout.WEST);
    p.add(jtf, BorderLayout.NORTH);
    jtf.setHorizontalAlignment(JTextField.LEFT);
    
    p.add(new JLabel("Enter Name"), BorderLayout.WEST);
    p.add(jtf1, BorderLayout.NORTH);
    jtf1.setHorizontalAlignment(JTextField.LEFT);
    
    p.add(new JLabel("Enter Phone"), BorderLayout.WEST);
    p.add(jtf2, BorderLayout.NORTH);//check!!!!
    jtf2.setHorizontalAlignment(JTextField.LEFT);
 

    setLayout(new BorderLayout());
    add(p, BorderLayout.NORTH);
    add(new JScrollPane(jta), BorderLayout.CENTER);
  
    jtf.addActionListener(new ButtonListener()); // Register listener
    jtf1.addActionListener(new ButtonListener()); // Register listener
    jtf2.addActionListener(new ButtonListener()); // Register listener


    setTitle("Client");
    setSize(500, 300);
    setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    setVisible(true); // It is necessary to show the frame here!

    try {
      // Create a socket to connect to the server
      Socket socket = new Socket("localhost", 8000);

      // Create an input stream to receive data from the server
      fromServer = new DataInputStream(
        socket.getInputStream());

      // Create an output stream to send data to the server
      toServer =
        new DataOutputStream(socket.getOutputStream());
    }
    catch (IOException ex) {
      jta.append(ex.toString() + '\n');
 
    }
  }

  private class ButtonListener implements ActionListener {
    public void actionPerformed(ActionEvent e) {
      try {
        // Get the name and phone from the text field
    	int n = Integer.parseInt(jtf.getText().trim());
        String name = (jtf1.getText().trim());
        String phone = (jtf2.getText().trim());

        
         toServer.writeInt(n);
         toServer.flush();

        // Send the name to the server
        toServer.writeUTF(name);
        toServer.flush();//Flushes this data output stream. This forces any buffered output bytes to be written out to the stream.
        // Send the phone to the server
        toServer.writeUTF(phone);
        toServer.flush();


        // Get name from the server
        String nm = fromServer.readUTF();
        // Get phone from the server
        String ph = fromServer.readUTF();


        // Display to the text area
        jta.append("Name is : " + nm + "\n");

        jta.append("Phone is : " + ph + "\n");


      }
      catch (IOException ex) {
        System.err.println(ex);
      }
    }
  }
}

