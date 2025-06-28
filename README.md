# Webserv

Voici les notes prises lors du projet webserv de l'école 42.

# C’est quoi un serveur web (HTTP) ?

Un serveur web HTTP (Hypertext Transfer Protocol), est un ordinateur qui stocke, gère et sert des pages web et d'autres ressources à des clients, tels que des navigateurs web.

Il joue un rôle essentiel dans le fonctionnement d'Internet et du World Wide Web.

Voici comment cela fonctionne :

- **Demande du client (request)** : Lorsqu'un utilisateur saisit une URL dans son navigateur ou clique sur un lien hypertexte, le navigateur envoie une demande (request) à un serveur web pour récupérer la page ou la ressource associée.
- **Traitement de la demande** : Le serveur web reçoit la demande et la traite. Il détermine quelle page ou ressource doit être renvoyée au client en fonction de l'URL demandée.
- **Renvoi de la réponse** (response): Une fois que le serveur web a déterminé la ressource à renvoyer, il génère une réponse (response) au format HTML, texte, image, ou tout autre format requis. Cette réponse est ensuite envoyée au navigateur du client.
- **Affichage côté client** : Le navigateur du client reçoit la réponse et l'affiche pour que l'utilisateur puisse la voir et interagir avec la page web.

![image_0](https://github.com/FXC-ai/webserv/blob/main/image.png)

Un serveur web utilise le protocole HTTP pour communiquer avec le navigateur du client. Ce protocole définit les règles et les conventions qui régissent la communication entre le client et le serveur, permettant ainsi le transfert de données, la demande de ressources, et bien d'autres fonctionnalités liées au web.

Il existe de nombreux serveurs web populaires, dont Apache, Nginx, Microsoft Internet Information Services (IIS), et LiteSpeed, qui sont utilisés pour héberger des sites web, des applications web et d'autres services en ligne. Chacun de ces serveurs web a ses propres caractéristiques et avantages, et le choix du serveur dépend souvent des besoins spécifiques du projet.

Bref. Assez de blabla. Nous on va construire notre propre serveur web avec les caractéristiques demandées par l’école 42. Donc pour mieux comprendre ce qu’on devra faire dans les détails, prenons un code simple d’un serveur web HTTP.

## Exemple de Serveur HTTP en C++

On a demandé à notre cher ami ChatGPT de nous coder un serveur web minimaliste en moins de 100 lignes (code à prendre avec des pincettes - on a connu des serveurs plus sécurisés - mais c’est juste pour comprendre les grandes lignes).

### Côté serveur

```c
#include <iostream>, <string>, <sstream>, <vector>, <unistd.h> // librairies classiques
#include <sys/socket.h>
#include <netinet/in.h>

const int PORT = 8080;
```

Comme d’hab, au début on définit les librairies qu’on utilise. J’ai regroupé les librairies “classiques” sur une première ligne pour gagner de la place - celles qui nous intéressent vraiment sont les deux suivantes. Ces bibliothèques fournissent des fonctionnalités pour gérer les connexions réseau et les opérations d'entrée/sortie.

On y définit aussi un port sur lequel le serveur va écouter les connexions entrantes.

→ Un port c’est un numéro qui aide les ordinateurs à acheminer les données vers la bonne application ou service. C’est en gros comme une porte numérotée sur un ordinateur. Alors alors, pour plus de détail, on peut dire que le port est le numéro de l’appartement. L’adresse IP est l’adresse de l’immeuble.

```c
std::string generateResponse() {
    std::string response = "HTTP/1.1 200 OK\r\n";
    response += "Content-Type: text/html\r\n\r\n";
    response += "<html><body><h1>Mon serveur web minimaliste</h1><p>Bienvenue !</p></body></html>";
    return response;
}
```

Cette fonction génère la réponse HTTP que le serveur enverra aux clients. Dans ce cas ici, elle crée une réponse HTTP simple avec le status "200 OK" (on verra plus loin ce que ça signifie vraiment), le type de contenu "text/html", et un message HTML de bienvenue.

```c
int main() {
    int serverSocket, clientSocket;
    struct sockaddr_in serverAddr, clientAddr;
    socklen_t clientLen = sizeof(clientAddr);
```

Ici, on crée 4 variables importantes :

- **serverSocket** & **clientSocket** : ce sont des descripteurs des sockets utilisés pour représenter la socket du serveur et la socket du client.

→ un *socket serveur* est un point d'écoute qui attend des connexions entrantes, alors qu'un *socket client* est un point de connexion qui se connecte à un serveur pour demander des services ou envoyer des données.

- **serverAddr** & **clientAddr** : ce sont des structures qui contiennent des informations sur les adresses IP et les ports du serveur et du client. Il existe différentes variantes de cette structure. sockaddre_in doit être utilisées avec la famille d’adresse AF_INET.
- **clientLen** : est une variable utilisée pour stocker la taille de la structure clientAddr. Cette taille est nécessaire lors de l'appel de la fonction accept() pour accepter une connexion entrante.

On va se contenter de ça pour le moment, et on verra à quoi elles servent au moment de les utiliser ensuite.

```c
// Créez une socket
    serverSocket = socket(AF_INET, SOCK_STREAM, 0);
    if (serverSocket < 0) {
        std::cerr << "Erreur lors de la création de la socket." << std::endl;
        return 1;
    }
```

Ce morceau de code crée une socket pour le serveur. C’est plus précisément la première ligne qui sera responsable de la création de la socket, à l’aide de la fonction socket().

→ En paramètre on lui donne : AF_INET qui indique que la socket utilisera le protocole IPv4. Donc que la socket sera utilisée pour la communication sur un réseau IPv4.

→ SOCK_STREAM indique que la socket sera de type flux (= utilisé pour des communications de type “flux de données”).

→ if (serverSocket < 0) : cette ligne vérifie si la création de socket a réussi (>0) ou fail (-1). Si ça foire, le code affiche un message d’erreur sur le fd d’erreur (std:cerr).

Il faut remplir la structure sockaddre_in dont le prototype est le suivant :

```yaml
#include <netinet/in.h>

struct sockaddr_in {
    short            sin_family;   // e.g. AF_INET
    unsigned short   sin_port;     // e.g. htons(3490)
    struct in_addr   sin_addr;     // see struct in_addr, below
    char             sin_zero[8];  // zero this if you want to
};

struct in_addr {
    unsigned long s_addr;  // load with inet_aton()
};
```

```c
serverAddr.sin_family = AF_INET;
serverAddr.sin_addr.s_addr = INADDR_ANY;
serverAddr.sin_port = htons(PORT);
```

Ici on configure les informations liées à l’adresse IP et au port du serveur. On va donner des informations supplémentaires :

- **serverAddr.sin_family** = AF_INET : comment avant, on lui dit que la famille d’adresses utilisée pour la config de l’adresse du serveur utilisera le protocole IPv4. AF_INET est toujours la famille d’adresse utilisée pour le protocole TCP/IP.
- **serverAddr.sin_addr.s_addr** = INADDR_ANY : spécifie l’adresse IP du serveur. INADDR_ANY signifie que le serveur sera accessible de tous les IP de l’ordi.
- **serverAddr.sin_port** = htons(PORT) : définit le numéro de port sur lequel le serveur écoutera les connexions. La fonction htons(PORT) est utilisée pour convertir le numéro de port de l'ordre octet local (big endian) à l'ordre octet réseau (network byte order). Cette conversion garantit que le numéro de port est correctement encodé pour la communication réseau.

→ Explication de ChatGPT : La conversion de l'ordre des octets (byte order en anglais) est un concept important dans la communication réseau. Cela concerne la manière dont les ordinateurs stockent les octets (8 bits) de données en mémoire. Il existe deux principaux ordres d'octets (petit-boutiste (little-endian)) & bing-endian.

```c
// Liez la socket au port

   if (bind(serverSocket, (struct sockaddr*)&serverAddr, sizeof(serverAddr)) < 0) {
        std::cerr << "Erreur lors de la liaison du serveur au port " << PORT << std::endl;
        return 1;
    }
```

La fonction *bind()* associe la socket du serveur à l'adresse et au port pour lequel le serveur écoutera les connexions entrantes. En d’autres termes, La fonction bind() sert à dire au serveur sur quel numéro de porte et à quelle adresse il doit attendre les connexions des clients.  Si la fonction bind foire (val < 0), un message d’erreur est envoyé sur la sortie d’erreur std::cerr.

```c
// Écoutez les connexions entrantes
    listen(serverSocket, 5);

    std::cout << "Serveur en cours d'exécution sur le port " << PORT << std::endl;
```

La ligne listen(serverSocket, 5) permet au serveur de commencer à écouter les connexions entrantes sur la socket. La valeur 5 indique le nombre maximum de connexions en attente qui seront autorisées en même temps. Le socket ne peut gérer qu’une connexion à la fois. Mais il peut en mettre un certain nombre en fil d’attente. Ici le la longueur maximale de la fil d’attente est 5. Les tentatives de connexion supplémentaires seront refusées.

Puis une fois que tout est mis en place, le serveur peut être lancé/exécuté. On affiche un message dans la console pour prévenir l’utilisateur.

```c
while (true) {
        // Acceptez une connexion entrante
        clientSocket = accept(serverSocket, (struct sockaddr*)&clientAddr, &clientLen);
 std::cout << "Nouvelle connexion acceptée." << std::endl;
	
        if (clientSocket < 0) {
            std::cerr << "Erreur lors de l'acceptation de la connexion." << std::endl;
            return 1;
        }

        char buffer[1024];

        ssize_t bytesRead = read(clientSocket, buffer, sizeof(buffer));

        if (bytesRead > 0) {
            std::string request(buffer, bytesRead);
            std::string response = generateResponse();

            // Envoyer la réponse au client
            send(clientSocket, response.c_str(), response.length(), 0);
    std::cout << "Réponse envoyée au client." << std::endl;
        }

        close(clientSocket);
 std::cout << "Connexion fermée." << std::endl;

    }

    close(serverSocket);
    return 0;
}
```

Cette boucle représente la partie principale du serveur. Elle est responsable de la gestion des connexions entrantes, de la lecture des demandes des clients, de la génération de réponses et de l'envoi de ces réponses aux clients.

Qu’est-ce qu’on y fait dans cette boucle exactement ? Et bien lorsqu’un client tente de se connecter, la fonction accept() est appelée et accepte une nouvelle connexion entrante. Elle renvoie un descripteur de la socket du client. Ce descripteur sera utilisé pour communiquer avec ce client spécifique. Si l’acceptation de la connexion se passe mal pour une raison ou une autre, on renvoie une erreur et le serveur s’arrête.

Si tout fonctionne, tu crées un buffer dans lequel on y stocke les données reçues du client (qui ont été envoyées par le client depuis sa socket). La fonction read() renvoie le nombre d'octets lus qui est stocké dans bytesRead (et si < 0, alors les données ont pas été lues avec succès).

Si on arrive bien à lire les données du client, elles sont converties en une chaîne de caractères pour faciliter le traitement grâce à request(buffer, bytesRead). Request est une variable qui contient la demande du client. Puis en suite une réponse du serveur, sous forme de chaîne de caractère, est générée en appelant la fonction qu’on a créé tout au début (generateResponse()). La réponse est envoyée au client grâce à la fonction send(). Une fois que la réponse a été envoyée au client, la socket cliente est fermée pour libérer les ressources.

La boucle continue ensuite à accepter de nouvelles connexions entrantes, à lire les demandes des clients, à générer des réponses et à envoyer ces réponses. Le serveur continue à fonctionner tant que la boucle while (true) est active. Lorsque vous décidez de fermer le serveur, par exemple en appuyant sur Ctrl+C, la boucle s'arrête, et le serveur se termine correctement en fermant également la socket du serveur (serverSocket).

Bref, maintenant qu’on a compris comment ça fonctionne côté serveur, voyons voir comment cela fonctionne côté client !

### Côté client

Quand je lancer le serveur, il faut que je tape ceci dans mon navigateur web :

**http://localhost:8080/**

En faisant ça, je deviens un client qui envoie une demande au serveur web en cours d'exécution sur le port 8080 de ma machine locale (localhost).

Lorsque j’accède à "[http://localhost:8080](http://localhost:8080/)", voici ce qu’il se passe :

- Mon navigateur envoie une demande HTTP GET au serveur sur le port 8080 de ma machine (localhost).
- Le serveur (le code ci-dessus) accepte cette demande, génère une réponse HTTP et renvoie cette réponse à mon navigateur.
- Mon navigateur reçoit la réponse du serveur et l'affiche dans la fenêtre du navigateur. Dans ce cas, la réponse est une page HTML avec le titre "Mon serveur web minimaliste" et un message de bienvenue. Visuelle, ça donne ça :
- 

![image_1](https://github.com/FXC-ai/webserv/blob/main/image(1).png)

Et du côté serveur, on obtient ces messages  :

![image_2](https://github.com/FXC-ai/webserv/blob/main/image(2).png)

On voit bien que tout a bien fonctionné comme prévu et le serveur continue de tourner :) Si un client lancer à nouveau une nouvelle requête (qu’il refresh la page web par exemple), une demande sera envoyée au serveur, qui sera acceptée, etc.

Une vidéo sympa pour revoir les bases une 10e fois :

![HTTP : comprendre l'essentiel en 4 minutes](https://www.youtube.com/watch?v=WGdOWtKL5nA)

## A propos du header et body

Maintenant que nous avons compris comment faire communiquer un serveur et son client et que nous sommes en mesure d'afficher une (magnifique) page web dans la fenêtre de notre navigateur préféré. Nous allons entrer un peu plus dans le détail des informations que le serveur envoie au client.

Dans cet exemple, nous allons envoyer une chaîne de caractères qui contient toutes les informations dont notre navigateur a besoin pour faire correctement son travail. Il s’agit du header et du body.

### Le header

Il contient a minima ces 3 informations :

- le protocol utilisé et sa version, le “status code” et le “status message” : HTTP/1.1 200 OK
- le type de contenu envoyé. Il varie si la page web contient du texte ou des images. Dans notre exemple : Content-Type: text/html
- le longueur de la chaîne de caractère envoyée : Content-Length: 315

Le header peut contenir d’autres informations comme la date et le nom du serveur.

Doit suivre ensuite une ligne séparatrice :

Schématiquement, cela correspond à l’image suivante :

![image_3](https://github.com/FXC-ai/webserv/blob/main/image(3).png)

### Le body

Enfin, on peut ajouter le code html de la page. Voici un exemple basique :

<!doctype html>

<html>

<head>

<title>Sever Strike Back</title>

<meta name="description" content="Our first page">

<meta name="keywords" content="html tutorial template">

</head>

<body>

<div>

Hello from the server side !

</div>

</body>

</html>

Comme précédemment, il suffit ensuite de se connecter au serveur via le navigateur à l’aide l’adresse : localhost:8888. 8888 est le numéro du port choisis pour tester le serveur (cf. code du serveur ci-dessous).

Et la magie opère, voici l’affichage de la page :

![image_4](https://github.com/FXC-ai/webserv/blob/main/image(4).png)

### Code complet :

```c
#include <iostream>
#include <stdio.h>
#include <sys/socket.h>
#include <unistd.h>
#include <stdlib.h>
#include <netinet/in.h>
#include <string.h>
#include <fstream>
#include <sstream>
#include <string>

int main()
{

   int sock_server;                                                            // creating socket
   int sock_client;
   struct sockaddr_in server_addr;                                             // creating address structure

   const int addr_len = sizeof(server_addr);

   const int PORT = 8888;                                                      // port number

   std::string index_html;

   std::ifstream file("index.html");
   std::ostringstream fileStream;

   fileStream << file.rdbuf();

   index_html = fileStream.str();

   std::string msg = "HTTP/1.1 200 OK\nContent-Type: text/html\nContent-Length: 315\r\n\r\n";

   msg = msg + index_html;

   std::cout << "content = " << msg.length() << std::endl;

   std::cout << msg << std::endl;

   sock_server = socket(AF_INET, SOCK_STREAM, 0);

   std::cout << "socket created" << std::endl;

   bzero(&server_addr, sizeof(server_addr));                      

   server_addr.sin_family = AF_INET;          
   server_addr.sin_addr.s_addr = htonl(INADDR_ANY);
   server_addr.sin_port = htons(PORT);

   bind(sock_server,(struct sockaddr *)&server_addr,sizeof(server_addr));

   listen(sock_server, 30);

   while (42)
   {
       sock_client = accept(sock_server, (struct sockaddr *)&server_addr, (socklen_t*)&addr_len);
       std::cout << "Client connected : " << sock_client << std::endl;

       write(sock_client, msg.c_str(), msg.length());

       char buffer[1024] = {0};

       read(sock_client, buffer, 1024);

       std::cout << "Received message : " << buffer << std::endl;
      
       close(sock_client);
   }
  
   return 0;
}
```

Contrairement au code précédent, le contenu du fichier index.html est copié dans la variable index_html et ajouté aux éléments du header à l’aide de l’objet de flux ifstream.

## Utilisation des sockets

### Qu’est ce qu'un socket ?

Un socket est une interface entre le programme et les protocoles de communication. C’est un mécanisme de communication bidirectionnel interprocessus. Une fois la connexion établie, le client et le serveur disposent chacun d’un descripteur (ou pseudo fichier) vers l'extrémité correspondante de la connexion. Ce descripteur est propre à chaque processus.

Point important : d’après chat gpt, le fonctionnement d’une socket ressemble beaucoup au fonctionnement d’un pipe. Le incomming buffer du client est le outcomming buffer du serveur et inversement. Ainsi lorsque le serveur “write” sur la socket il ne risque pas d’overwrite les donnés écrite par le client sur cette même socket.

Les codes précédents fonctionnent parfaitement avec un seul client. La boucle utilisée côté serveur attends la demande d’un client en écoutant sur un socket spécifique :

```c
listen(sock_server, 30);

while (42)
   {
       sock_client = accept(sock_server, (struct sockaddr *)&server_addr, (socklen_t*)&addr_len);
       std::cout << "Client connected : " << sock_client << std::endl;

       write(sock_client, msg.c_str(), msg.length());

       char buffer[1024] = {0};

       read(sock_client, buffer, 1024);

       std::cout << "Received message : " << buffer << std::endl;
      
       close(sock_client);
   
}
```

La fonction accept attends la connexion d’un client. Dès que le client envoie une requête au serveur, elle écrit la réponse dans le socket du client. Dans un second temps elle lit le contenu du socket du client puis affiche ce qu’elle a lu.

Côté client voici le code :

```c
#include <iostream>
#include <stdio.h>
#include <sys/socket.h>
#include <stdlib.h>
#include <unistd.h>
#include <netinet/in.h>
#include <string.h>
#include <arpa/inet.h>
#include <unistd.h>

#define PORT 8888

int main()
{
   int     sock_client;   
   struct  sockaddr_in server_addr;   
   const   std::string message = "Hello from client";
   char    tempBuffer[1024] = {0};

   sock_client = socket(AF_INET, SOCK_STREAM, 0);

   std::cout << "sock_client = " << sock_client <<std::endl;

   server_addr.sin_family = AF_INET;
   server_addr.sin_port = htons(PORT);

   server_addr.sin_addr.s_addr = inet_addr("127.0.0.1");

   int connect_status = connect(sock_client, (struct sockaddr *)&server_addr, sizeof(server_addr));

   std::cout << "Connection status = " << connect_status << std::endl;

   int send_status = send(sock_client, message.c_str(), message.length(), 0);

   std::cout << "Send status " << send_status << std::endl;

   int read_status = read(sock_client, tempBuffer, 1024);

   std::cout << "read status " << read_status << std::endl;

   std::cout << "Received message : " << std::endl;
  
   std::cout << tempBuffer << std::endl;

   return 0;
}
```

Après la connexion au serveur, le client écrit sur la socket le contenu de la variable message. Il s’agira alors de sa requête et le serveur write sur cette même socket le contenu la page à afficher.

### Le problème

Maintenant imaginons que pour une raison ou une autre le client monopolise la connexion au serveur via le port autorisé. C’est le cas si on ajoute une fonction sleep() juste après la connexion. Dans ce cas, aucun autre client ne peut utiliser ce port tant que le premier client n’a pas terminé… C’est pourquoi les fonctions select et poll ont été inventées.

Les sockets doivent être fermées ou maintenus ouvertes. Lors d’un test avec siege, les sockets doivent être fermées après chaque traitement de requête. Mais il peut être utile de garder la socket ouverte même après avoir traité la requête.

# Overview du projet

Comment qui marche ce WebServ ?
Tous d'abords mettons les choses au clair notre serveur doit pouvoir écouter sur plusieurs ports. C'est à dire qu'il doit être en mesure de servir plusieurs sites en même temps. Ainsi selon la requête du Client, il pourra envoyer le résultat en provenance de tel ou tel hôte. En fait derrière chaque port sur lequel on écoute se cache un serveur name que l'on peut assimiler à un nom de domaine. Donc basiquement chaque port écouté représente un "site internet" à servir.

1) D'abord Victor fournit un vecteur contenant des objets ServerConfig. Chaque objet contient une config serveur unique. Dans chaque config on a un Port a écouter et un server name.
2) Je me sert de chaque objet contenu dans ce vecteur pour écouter sur chaque port et leur affilié une socket d'écoute.
3) Chaque fois qu'un client se connectera un l'une de ces sockets d'écoute, je crée un objet client qui contiendra la requête et la configuration du serveur avec lequel le client voudra communiquer. A ce moment là la response du Client sera vide.
4) J'envoie ce client à Laura qui analyse la requête à la lumière de la configuration du serveur et construit la réponse en fonction. C'est à ce moment qu'il faudra potentiellement utiliser les CGI si la page à laquelle le client veut accéder est dynamique.
5)Laura me renvoie l'objet client avec sa response dûment complétée.

6) J'envoie la réponse au client ...

![Untitled](https://github.com/FXC-ai/webserv/blob/main/Untitled.png)

# Config

Exemple de fichier de configuration :

```
server:
server_name: localhost
port: 8888
ip: 127.0.0.1
max_body_size: 300000

default_file: /default.html

# Default error pages
error_page:

# Root directory for the server
root: web/website0

location:
	path: /
	methods: GET, POST
	redirect: /index.html

location:
	path: /uploads
	methods: POST, DELETE, GET
	directory_listing: on

location:
	path: /home.html
	methods: POST, GET
	redirect: /index.html

location:
	path: /img
	methods: POST, GET
	directory_listing: off
	redirect: /index.html

location:
	path: /test
	methods: POST, GET
	directory_listing: off

```

# Request / Response

## Principaux codes d’erreur

![image.webp](https://raw.githubusercontent.com/FXC-ai/webserv/refs/heads/main/image.webp)

## Exemples de requête

Request = GET / HTTP/1.1
Host: localhost:8888
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/117.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1

GET /teststyle.css HTTP/1.1
Host: localhost:8888
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:120.0) Gecko/20100101 Firefox/120.0
Accept: text/css,/;q=0.1
Accept-Language: fr,fr-FR;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Referer:http://localhost:8888/

Sec-Fetch-Dest: style
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: same-origin

GET /favicon.ico HTTP/1.1
Host: localhost:8888
Connection: keep-alive
sec-ch-ua: "Not.A/Brand";v="8", "Chromium";v="114", "Google Chrome";v="114"
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.0.0 Safari/537.36
sec-ch-ua-platform: "macOS"
Accept: image/avif,image/webp,image/apng,image/svg+xml,image/*,*/*;q=0.8
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: no-cors
Sec-Fetch-Dest: image
Referer: http://localhost:8888/
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9

Remarques :

- **Chaque ligne d’une requête se termine par \r\n !**
- La requête peut contenir un body lorsqu’on utiliser la méthode POST comme par exemple lors de l’envoie d’un formulaire. Ce body se trouve après une ligne vide : **\r\n\r\n**.
- Voici un exemple de body d’une requête provenant d’un formulaire avec un seul champ appelé “VoilaLesDatasDuUser” : VoilaLesDatasDuUser=Hello+World
- Accept définis le type MIME, il en existe beaucoup, cela correspond au type du fichier concerné par la requête et par la réponse (https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)

## Fonctionnement général

Pour une seule page, il y a plusieurs requêtes : une requête pour le html, une requête pour le css et une requête pour chaque fichier supplémentaires comme les images par exemple…

**Attention le schema n’est pas juste ! En cas de redirection il faut renvoyer un code 301 et faire en sorte que le navigateur renvoie une requête sur la page à rediriger.**

![canvas.webp](https://raw.githubusercontent.com/FXC-ai/webserv/refs/heads/main/canvas.webp)

# Sources

[GitHub - Kaydooo/Webserv_42: 42 Webserv](https://github.com/Kaydooo/Webserv_42/tree/main)

[GitHub - llefranc/42_webserv: School projet : implement a server handling HTTP protocol  using UNIX socket.](https://github.com/llefranc/42_webserv)

[HTTP Server: Everything you need to know to Build a simple HTTP server from scratch](https://medium.com/from-the-scratch/http-server-what-do-you-need-to-know-to-build-a-simple-http-server-from-scratch-d1ef8945e4fa)

[Example: Nonblocking I/O and select()](https://www.ibm.com/docs/en/i/7.2?topic=designs-example-nonblocking-io-select)

[C++ Web Server from Scratch | Part 1: Creating a Socket Object](https://www.youtube.com/watch?v=YwHErWJIh6Y)

Pour comprendre l’implémentation des sockets

[Building a simple server with C++](https://ncona.com/2019/04/building-a-simple-server-with-cpp/)

[C++ Web Programming](https://www.tutorialspoint.com/cplusplus/cpp_web_programming.htm)

[Mettre en place un serveur Web (21/28) : Nginx](https://www.youtube.com/watch?v=YD_exb9aPZU)

Comprendre et mettre en place un serveur Nginx

https://du-isn.gricad-pages.univ-grenoble-alpes.fr/2-sr/Reseaux/3--cours_reseaux--sockets_java.pdf

https://www.lirmm.fr/~bosio/GMEE115/04-sockets.pdf

[Socket Programming in C/C++: Handling multiple clients on server without multi threading - GeeksforGeeks](https://www.geeksforgeeks.org/socket-programming-in-cc-handling-multiple-clients-on-server-without-multi-threading/?ref=gcse)

[How to perform HTTP load testing using Siege - Interserver Tips](https://www.interserver.net/tips/kb/http-load-testing-siege/)

[Waiting for I/O (The GNU C Library)](https://www.gnu.org/software/libc/manual/html_node/Waiting-for-I_002fO.html)

[Blog Stéphane Bortzmeyer: RFC 7230: Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing](https://www.bortzmeyer.org/7230.html)

[Network Programming - [3] bind() vs connect() and accept() method](http://ryansiroiro.blogspot.com/2018/07/network-programming-3-bind-vs-connect.html)

[HTTP Status Codes: A Complete List + Explanations](https://www.semrush.com/blog/http-status-codes/?kw=&cmp=FR_SRCH_DSA_Blog_EN&label=dsa_pagefeed&Network=g&Device=c&utm_content=676606895405&kwid=dsa-2185834089056&cmpid=18361911540&agpid=158109740287&BU=Core&extid=105137755402&adpos=&gad_source=1&gclid=Cj0KCQiA67CrBhC1ARIsACKAa8SAfW9aitNveWWwlb7ocl4Z41YMVkKFzLZcGv9dIkBG7iGsuDSJyEMaAkfyEALw_wcB)

[Passing arguments to the Python script from the command line.](https://datageeks.medium.com/passing-parameters-to-the-python-script-from-the-command-line-139a9fc94ee)

https://www.youtube.com/watch?v=tkfVQK6UxDI

[Sec-Fetch-Dest - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Sec-Fetch-Dest)
