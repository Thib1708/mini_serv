# mini_serv
``` C
/* ----------- GIVE BY THE SUBJECT ----------- */
#include <errno.h>
#include <string.h>
#include <unistd.h>
#include <netdb.h>
#include <sys/socket.h>
#include <netinet/in.h>
/* ------------------------------------------- */
#include <stdio.h>
#include <stdlib.h>


void sendToAll(int client, int max_fd, char *message, fd_set w_socks) {
	for (int fd = 2; fd <= max_fd; fd++) {
		if (FD_ISSET(fd, &w_socks) && fd != client) {
			send(fd, message, strlen(message), 0);
		}
	}
}


int main(int argc, char **argv) {
	/* ----------- GIVE BY THE SUBJECT ----------- */
	int sockfd, connfd;
	struct sockaddr_in servaddr;
	/* ------------------------------------------- */
	fd_set a_socks, r_socks, w_socks;
	int clients[42000];
	int next_id = 0;
	char message[400200];

	if (argc != 2) {
		write(2, "Wrong number of arguments\n", strlen("Wrong number of arguments\n"));
		exit(1);
	}

	/* ----------- GIVE BY THE SUBJECT ----------- */
	// socket create and verification 
	sockfd = socket(AF_INET, SOCK_STREAM, 0); 
	if (sockfd == -1) { 
		write(2, "Fatal error\n", strlen("Fatal error\n")); 
		exit(1); 
	} 
	bzero(&servaddr, sizeof(servaddr));

	// assign IP, PORT 
	servaddr.sin_family = AF_INET; 
	servaddr.sin_addr.s_addr = htonl(2130706433); // 127.0.0.1
	servaddr.sin_port = htons(atoi(argv[1])); 
  
	// Binding newly created socket to given IP and verification 
	if ((bind(sockfd, (const struct sockaddr *)&servaddr, sizeof(servaddr))) != 0) { 
		write(2, "Fatal error\n", strlen("Fatal error\n")); 
		exit(1); 
	} 
	if (listen(sockfd, 4096) != 0) {
		write(2, "Fatal error\n", strlen("Fatal error\n")); 
		exit(1); 
	}
	/* ------------------------------------------- */

	FD_ZERO(&a_socks);
	FD_SET(sockfd, &a_socks);
	
	int max_fd = sockfd;
	while(1)
	{
		r_socks = w_socks = a_socks;

		if (select(max_fd + 1, &r_socks, &w_socks, NULL, NULL) < 0) {
			continue; 
		}
		if (FD_ISSET(sockfd, &r_socks)) {
			connfd = accept(sockfd, NULL, NULL);
			if (connfd < 0) { 
				write(2, "Fatal error\n", strlen("Fatal error\n")); 
				exit(1); 
			}
			FD_SET(connfd, &a_socks);
			sprintf(message, "server: client %d just arrived\n", next_id);
			sendToAll(connfd, max_fd, message, w_socks);
			clients[connfd] = next_id++;
			if (connfd > max_fd) {
				max_fd = connfd;
			}
			continue;
		}

		for (int fd = 2; fd <= max_fd; fd++) {

			if (FD_ISSET(fd, &r_socks)) {
				char buffer[400000];
				bzero(buffer, sizeof(buffer));
				int bytes_read = 1;

				while (bytes_read == 1 && buffer[strlen(buffer) - 1] != '\n') {
					bytes_read = recv(fd, buffer + strlen(buffer), 1, 0);
				}
				if (bytes_read <= 0) {
					sprintf(message, "server: client %d just left\n", clients[fd]);
					sendToAll(fd, max_fd, message, w_socks);
					close(fd);
					FD_CLR(fd, &a_socks);
				} else {
					sprintf(message, "client %d: %s", clients[fd], buffer);
					sendToAll(fd, max_fd, message, w_socks);
				}
			}
		}
	}
}
```
