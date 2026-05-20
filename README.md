package Network;

public enum CommandType {
    HELLO, EXIT, CHAT;
}



package Network;

import lombok.*;

import java.io.Serializable;

@AllArgsConstructor
@NoArgsConstructor
@Setter
@Getter
@ToString
@Builder
public class Request implements Serializable {
    private CommandType commandType;
    private Object data;
}




package Network;

import lombok.*;

import java.io.Serializable;

@AllArgsConstructor
@NoArgsConstructor
@Setter
@Getter
@ToString
@Builder
public class Response implements Serializable {
    private static final long serialVesionUID=1L;
    private boolean susccess;
    private Object data;
    private String message;

}

package Network;

import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.net.Socket;
import java.util.Scanner;

public class Client {
    public static void main(String[] args) {
        try(
                Socket socket= new Socket("localhost", 9090);
                ObjectOutputStream out = new ObjectOutputStream(socket.getOutputStream());
                ObjectInputStream in = new ObjectInputStream(socket.getInputStream());
                Scanner sc= new Scanner(System.in);
                ){
            int choice = 0;
            while (true){
                System.out.println("Nhập vào số");
                choice= sc.nextInt();
                Request request= new Request();
                switch (choice){
                    case 1:
                        request.setCommandType(CommandType.HELLO);
                }
                out.writeObject(request);
                out.flush();
                Response response= (Response) in.readObject();
                System.out.println(response);
            }
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }
}





package Network;

import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;


public class Server {
    public static void main(String[] args) {
        ExecutorService pool = Executors.newFixedThreadPool(10);
        try(
                ServerSocket serverSocket = new ServerSocket(9090);

        ){
            System.out.println("Sytem is ready");
            while (true){
                Socket socket = serverSocket.accept();
                ClientHandler handler = new ClientHandler(socket);
                pool.submit(handler);
            }
        }catch (Exception ex){
            throw new RuntimeException(ex);
        }
    }
}
class ClientHandler implements Runnable{
    private  Socket socket;

    public ClientHandler(Socket socket) {
        this.socket = socket;
    }

    @Override
    public void run(){
        try(
                ObjectOutputStream objectOutputStream = new ObjectOutputStream(socket.getOutputStream());
                ObjectInputStream objectInputStream = new ObjectInputStream(socket.getInputStream());
        ){
            while (true){
                Response response = new Response();
                Request request = (Request) objectInputStream.readObject();
                CommandType commandType = request.getCommandType();
                switch (commandType){
                    case HELLO -> System.out.println("Hello");

                }


                objectOutputStream.writeObject(response);
                objectOutputStream.flush();
            }
        }catch (Exception e){
            throw new RuntimeException(e);
        }
    }

}









