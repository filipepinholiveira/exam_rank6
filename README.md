# exam_rank6





Objetivo do Programa

Este programa implementa um servidor de chat simples que usa Sockets e select() para gerenciar múltiplos clientes simultaneamente. O servidor permite que vários clientes se conectem, enviem mensagens uns para os outros, e recebe uma notificação quando um cliente entra ou sai.

Fluxo Principal

1. Criação do servidor: O servidor é inicializado usando socket(), bind() e listen().

2. Conexão de clientes: O servidor espera que clientes se conectem usando select() para aguardar eventos (como um novo cliente tentando se conectar ou dados de um cliente existente).

3. Troca de mensagens: Quando um cliente envia uma mensagem, o servidor a distribui para todos os outros clientes conectados.

4. Saída de clientes: Quando um cliente se desconecta, o servidor notifica os outros clientes que ele saiu.


# #################################################################################################################### #


Explicação Detalhada do Código

1.      Cabeçalhos e Definições:

    - O código usa bibliotecas padrão como stdio.h, unistd.h, stdlib.h, string.h, netinet/in.h, e sys/select.h.

    - As mensagens de erro, chegada e saída dos clientes são definidas como constantes de string:


            #define WRONG "Wrong number of arguments\n"
            #define FATAL "Fatal error\n"
            #define ARRIVED "server: client %d just arrived\n"
            #define LEFT "server: client %d just left\n"
            #define CLTMSG "client %d: %s"



# #################################################################################################################### #


2.      Função exiterror:

    Esta função é chamada quando ocorre um erro. Ela fecha o servidor e imprime a mensagem de erro.


            void exiterror(const char *msg) 
            {
                if (server > 2) close(server);  // Fecha o servidor, se aberto
                write(2, msg, strlen(msg));     // Imprime a mensagem de erro
                exit(1);                        // Finaliza o programa
            }


# #################################################################################################################### #


3.      Função sendresponses:

    - Esta função é responsável por enviar mensagens para todos os clientes conectados, exceto o cliente que gerou a mensagem (o ownfd).

    - Ele percorre todos os clientes e envia a mensagem usando send().

            void sendresponses(const int ownfd) 
            {
            for (int fd = 2; fd <= last; ++fd) 
                {
                    if (fd != ownfd && FD_ISSET(fd, &sendtime)) 
                    {
                    if (send(fd, str, strlen(str), 0) < 0) exiterror(FATAL);
                    }
                }
            bzero(&str, sizeof(str));  // Limpa o buffer de mensagens
            }


# #################################################################################################################### #


4.      Função Principal main:


    - Inicialização do Servidor: O servidor é configurado para escutar na porta passada como argumento. Se o servidor não conseguir configurar o socket ou escutar, ele chama exiterror() para exibir um erro e sair.


            server = socket(AF_INET, SOCK_STREAM, 0);
            if (server < 0) exiterror(FATAL);
            if ((bind(server, (const struct sockaddr *)&servaddr, sizeof(servaddr))) != 0) exiterror(FATAL);
            if (listen(server, 10) != 0) exiterror(FATAL);


    - Configuração de select: O select() é usado para monitorar múltiplos descritores de arquivos (clientes conectados e o servidor) e saber quando há dados disponíveis para leitura ou quando um novo cliente está tentando se conectar.


            FD_ZERO(&requests);
            FD_SET(server, &requests);
            last = server;


    - Aceitar Novos Clientes: Quando um novo cliente se conecta, o servidor aceita a conexão usando accept(). O ID do cliente é atribuído e ele é registrado para receber mensagens.


            client = accept(server, (struct sockaddr *)&cli, &len);
            if (client < 0) exiterror(FATAL);
            sprintf(str, ARRIVED, id);  // Mensagem de chegada
            clients[client] = id++;     // Atribui um ID único ao cliente
            FD_SET(client, &requests);  // Registra o cliente
            sendresponses(client);      // Envia mensagem para os outros clientes


    - Gerenciamento de Mensagens: Quando um cliente envia uma mensagem, o servidor recebe os dados com recv(), formata a mensagem e a distribui para todos os outros clientes usando a função sendresponses().


            while (r == 1 && buff[strlen(buff) - 1] != '\n') r = recv(fd, buff + strlen(buff), 1, 0);
            if (r <= 0) 
            {
                sprintf(str, LEFT, clients[fd]);  // Cliente saiu
                FD_CLR(fd, &requests);
                close(fd);
            } 
            else 
            {
                sprintf(str, CLTMSG, clients[fd], buff);  // Formata a mensagem
            }
            sendresponses(fd);  // Envia para todos os outros clientes


    - Cliente Saiu: Quando um cliente se desconecta, o servidor remove o cliente da lista e envia uma mensagem dizendo que ele saiu.

            sprintf(str, LEFT, clients[fd]);
            FD_CLR(fd, &requests);
            close(fd);





