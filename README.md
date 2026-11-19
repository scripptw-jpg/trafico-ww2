import socket
import threading
import time

# ============ SERVIDOR ============
def iniciar_servidor():
    # 1. Crear el socket TCP
    servidor_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    servidor_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

    # 2. Vincular a IP y puerto
    host = '127.0.0.1'
    puerto = 8080
    servidor_socket.bind((host, puerto))

    # 3. Poner en escucha
    servidor_socket.listen(1)
    print(f"[*] Servidor escuchando en http://{host}:{puerto}")
    print("[*] Esperando a que ocurra el TCP 3-way handshake...")

    while True:
        # 4. Aceptar la conexión (Aquí se completa el Handshake TCP a bajo nivel)
        conexion_cliente, direccion_cliente = servidor_socket.accept()
        print(f"\n[+] ¡Conexión TCP establecida con éxito desde {direccion_cliente}!")

        # 5. Recibir los datos de la solicitud (Tráfico WWW simulado)
        peticion = conexion_cliente.recv(1024).decode('utf-8')
        print(f"--- Solicitud recibida del cliente ---\n{peticion}")

        # 6. Enviar una respuesta simulada de la World Wide Web (HTML)
        respuesta_html = (
            "HTTP/1.1 200 OK\r\n"
            "Content-Type: text/html; charset=utf-8\r\n\r\n"
            "<html><body><h1>¡Hola desde el servidor TCP simulado de la WWW!</h1></body></html>"
        )
        conexion_cliente.sendall(respuesta_html.encode('utf-8'))

        # 7. Cerrar la conexión
        conexion_cliente.close()
        print("[*] Conexión cerrada por el servidor.")

# ============ CLIENTE ============
def iniciar_cliente():
    # 1. Crear el socket TCP del cliente
    cliente_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

    host = '127.0.0.1'
    puerto = 8080

    print(f"[*] Intentando conectar con el servidor en {host}:{puerto} (Iniciando TCP)")

    # 2. Conectar al servidor (Envía SYN -> Recibe SYN-ACK -> Envía ACK)
    cliente_socket.connect((host, puerto))
    print("[+] ¡Conexión TCP establecida con el servidor!")

    # 3. Simular una petición HTTP de la World Wide Web
    peticion_http = "GET / HTTP/1.1\r\nHost: localhost\r\nConnection: close\r\n\r\n"
    print("[*] Enviando solicitud HTTP a través del canal TCP seguro...")
    cliente_socket.sendall(peticion_http.encode('utf-8'))

    # 4. Recibir la respuesta
    respuesta = cliente_socket.recv(4096).decode('utf-8')
    print(f"\n--- Respuesta recibida del servidor ---\n{respuesta}")

    # 5. Cerrar el socket
    cliente_socket.close()
    print("[*] Cliente desconectado.")

# ============ UNIR LOS DOS ============
if __name__ == "__main__":
    # Prende el servidor en segundo plano
    hilo_servidor = threading.Thread(target=iniciar_servidor, daemon=True)
    hilo_servidor.start()

    # Espera un poquito a que el servidor prenda
    time.sleep(1)

    # Ahora corre el cliente
    iniciar_cliente()

    print("\n[*] Prueba terminada.")
  
