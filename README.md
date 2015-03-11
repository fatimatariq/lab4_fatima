# package lab4;

import java.util.*;
import java.io.*;
import java.net.*;            //importing packeges



class CHandler extends Thread {
  private Socket socket; 

  
  public CHandler(Socket s) {
    socket=s;
    start();
  }

 
  public void run() {
    try {

      
      BufferedReader in=new BufferedReader(new InputStreamReader(
        socket.getInputStream()));
      PrintStream out=new PrintStream(new BufferedOutputStream(
        socket.getOutputStream()));

      // Read filename from first input line "GET /filename.html ..."
      // or if not in this format, treat as a file not found.
      String s=in.readLine();
      System.out.println(s);  

      String filename="";
      StringTokenizer st=new StringTokenizer(s);
      try {

       
        if (st.hasMoreElements() && st.nextToken().equalsIgnoreCase("GET")
            && st.hasMoreElements())
          filename=st.nextToken();
        else
          throw new FileNotFoundException();  // Bad request

        
        if (filename.endsWith("/"))
          filename+="fatima_lab4.html";

        // Remove leading / from filename
        while (filename.indexOf("/")==0)
          filename=filename.substring(1);

        
        filename=filename.replace('/', File.separator.charAt(0));

        
        if (filename.indexOf("..")>=0 || filename.indexOf(':')>=0
            || filename.indexOf('|')>=0)
          throw new FileNotFoundException();

       
        if (new File(filename).isDirectory()) {
          filename=filename.replace('\\', '/');
          out.print("HTTP/1.0 301 Moved "+
            "Location: /"+filename+"/\r\n");
          out.close();
          return;
        }

       
        InputStream f=new FileInputStream(filename);

       
        String mimeType="text/plain";
        if (filename.endsWith(".html") || filename.endsWith(".htm"))
          mimeType="text/html";
        else if (filename.endsWith(".jpg") || filename.endsWith(".jpeg"))
          mimeType="image/jpeg";
        else if (filename.endsWith(".gif"))
          mimeType="image/gif";
        out.print("HTTP/1.0 200 OK\r\n"+
          "Content-type: "+mimeType+"\r\n");

        // Send file contents to client, then close the connection
        byte[] a=new byte[4096];
        int n;
        while ((n=f.read(a))>0)
          out.write(a, 0, n);
        out.close();
      }
      catch (FileNotFoundException x) {
        out.println("HTTP/1.0 404 Not Found\r\n"+
          "Content-type: text/html\r\n"+
          "<html><head></head><body>"+filename+" not found</body></html>\n");
        out.close();
      }
    }
    catch (IOException x) {
      System.out.println(x);
    }
  }
}

public class lab4 {
	  private static ServerSocket serverSocket;

	  public static void main(String[] args) throws IOException {
	    serverSocket=new ServerSocket(5005);  // Start, listen on port 5008
	    while (true) {
	      try {
	        Socket s=serverSocket.accept();  // Wait for a client to connect
	        new CHandler(s);  // Handle the client in a separate thread
	      }
	      catch (Exception x) {
	        System.out.println(x);
	      }
	    }
	  }
	}
