# CWEBSERVER
This is C based Webserver implemented by using only the standard libraries.

Technology Used:C

Operating System:Linux

The server runs on port number 8887 on Linux and includes features like handling HTTP GET/POST request ,handling stattic request of content types(txt, JSON, html, jpg, zip. rar, pdf, php etc.), dynamic requests (update database etc.) sending proper HTTP error codes and also provide facilty to set/get Cookie/Session.

## Benifits of using this Application:
1.It can Handle multiple requests at same time.
2.It is much faster than java/python based Webserver.
3.It is open source you can also provide some other functionalities.
4.Code is Easy to Understand.

I thought it would be useful for you if I share the code.
## Code
```c
#include<netinet/in.h>   
#include<stdio.h>       
#include<sys/socket.h>    
#include<sys/stat.h>    
#include<sys/types.h>    
#include<unistd.h> 
#include <stdarg.h>
#include<malloc.h>
#include<stdlib.h>
#include<string.h>
#include<dirent.h>
#define PORT 8887
char cwd[1000];
char sites[1000][50];
struct Pair
{
char attribute[50];
char value[50];
struct Pair *next;
};
struct Socket
{
int serverSocket;
struct sockaddr_in address;
};
struct Cookie
{
char cookieName[50];
char cookieValue[50];
char cookieExpiry[50];
struct Cookie *next;
};
struct Response
{
char  contentType[1000];
struct Cookie *cookie;
int httpCode;
int clientSocket;
};
struct Request
{
char siteName[50];
char url[100];
struct Cookie *cookie; 
struct Pair *pair;
char *(*p2f)(char ** , char ** ,char *);
};

typedef void (*usersFunction)(struct Request,struct Response);
typedef usersFunction (*mapFunction)(char *);
mapFunction requestProcessor;
void  getSession(struct Request *r,char *sessionId)
{
strcpy(sessionId," ");
struct Cookie *c;
c=r->cookie;	
while(c!=NULL)
{
if(strcmp(c->cookieName,"sessionId")==0)
{
strcpy(sessionId,c->cookieValue);
return;
}
c=c->next;
}
}
void  getCookie(struct Request *r ,char * cookieName,char * cookieValue)
{
strcpy(cookieValue," ");
struct Cookie *c;
c=r->cookie;
while(c!=NULL)
{
if(strcmp(c->cookieName,cookieName)==0)
{
strcpy(cookieValue,c->cookieValue);
return;
}
c=c->next;
}
}
void setContentType(struct Response *rs,char * contentType)
{
strcpy(rs->contentType,contentType);
}
void setHttpCode(struct Response *rs,int httpCode)
{
rs->httpCode=httpCode;
}
void setSession(struct Response *rs,char * sessionId)
{
struct Cookie *c,*cc;
c=(struct Cookie *)malloc(sizeof(struct Cookie));
strcpy(c->cookieName,"sessionId");
strcpy(c->cookieValue,sessionId);
strcpy(c->cookieExpiry," ");
c->next=NULL;
if(rs->cookie==NULL)
{
rs->cookie=c;
return;
}
cc=rs->cookie;
while(cc->next!=NULL)cc=cc->next;
cc->next=c;
}
void setCookie(struct Response *rs,char *cookieName,char *cookieValue,char *cookieExpiry)
{
struct Cookie *c,*cc;
c=(struct Cookie *)malloc(sizeof(struct Cookie));
strcpy(c->cookieName,cookieName);
strcpy(c->cookieValue,cookieValue);
strcpy(c->cookieExpiry,cookieExpiry);
c->next=NULL;
if(rs->cookie==NULL)
{
rs->cookie=c;
return;
}
cc=rs->cookie;
while(cc->next!=NULL)cc=cc->next;
cc->next=c;
}
void get(struct Request rq,char * str, ...)
{
struct Pair *pair;
char format[100];
char C[100][200];
int j=0,num=0,i=0,k=0;
char value[100];
va_list valist;
for(j=0;str[j]!='\0';j++)
{
if(str[j]==' ')continue;
if(str[j]=='%')num++;
j++;
if(str[j]=='{')
{
j++;
k=0;
while(str[j]!='}')
{
C[i][k]=str[j];
j++;
k++;
}
j++;
format[i]=str[j];
C[i][k]='\0';
i++;
}
}
format[i]='\0'; 
va_start(valist,num ); 
for (i = 0; i < num; i++)
{
pair=rq.pair;
while(pair!=NULL)
{
if(strcmp(pair->attribute,C[i])==0)
{
strcpy(value,pair->value);
break;
}
pair=pair->next;
}
if(pair==NULL)
{
printf("%s Attribute not exists\n",C[i]);
va_arg(valist,void *);
}
else
{
if(format[i]=='s')
{
sprintf(va_arg(valist,char *),"%s",value);
}
else if(format[i]=='d')
{
*(va_arg(valist,int *))=atoi(value);
}
else if(format[i]=='c')
{
*(va_arg(valist,char *))=value[0];
}
else if(format[i]=='f')
{
*(va_arg(valist,double *))=atof(value);
}      
else if(format[i]=='l')
{
*(va_arg(valist,long *))=atol(value);
}  
}
}
va_end(valist); 
}
void mapper(mapFunction k)
{
requestProcessor=k;
}
void sendHeader(struct Response rs)
{
char httpResponse[5000];
char contentType[100];
char cookies[1000];
char header[1000];
struct Cookie *c;
int i=0;
if(rs.httpCode==200)
{
sprintf(httpResponse,"HTTP/1.1 200 OK\n");
}
if(rs.httpCode==404)
{
sprintf(httpResponse,"HTTP/1.1 404 Not Found\n");
}
sprintf(contentType,"Content-Type: %s\n\n",rs.contentType);
cookies[0]='\0';
c=rs.cookie;
while(c!=NULL)
{
sprintf(cookies,"%sSet-Cookie:%s=%s;Expires=%s\n",cookies,c->cookieName,c->cookieValue,c->cookieExpiry);
c=c->next;
}
sprintf(header,"%s%s%s",httpResponse,cookies,contentType);
write(rs.clientSocket,header,strlen(header)); 
printf("Header:%s\n",header);
}
void pout(struct Response rs,char * str)
{
write(rs.clientSocket,str,strlen(str));
}
void populateDS()
{
int i,u=0,k;
DIR *dir;
DIR *dir1;
char path[1000];
struct dirent *ent;
if ((dir = opendir (cwd)) != NULL)
{
while ((ent = readdir (dir)) != NULL) 
{
k=sprintf(path,"%s//%s",cwd,ent->d_name);
if ((dir1 = opendir (path)) != NULL)
{
strcpy(sites[u],ent->d_name);
u++;
}
}
closedir (dir); 
}
else {
printf("//%s path not exists\n",cwd);
return ;
}
}
void sendResponse(struct Request rq,struct Response rs)
{
char siteName[1000];
char url[1000];
char content[10000];
int u=0;
char c;
int k=0;
char filePath[1000];
char * ext;
FILE *file;
int bufferSize=1024;
char buffer[bufferSize];
strcpy(url,rq.url);
strcpy(siteName,rq.siteName);
for(u=0;sites[u][0]!='\0';u++)
{
if(strcmp(siteName,sites[u])==0)
{
break;
}
}
if(sites[u][0]!='\0')
{
sprintf(filePath,"%s//%s//%s",cwd,siteName,url);
file=fopen(filePath,"r");
if(file==NULL){
rs.httpCode=200;
sprintf(rs.contentType,"text/html");
sendHeader(rs);
pout(rs,"<html><body><H1>404 Error</H1></body></html>");
}
else
{
ext = strrchr(filePath,'.');
if(strcmp(ext,".txt")==0)sprintf(rs.contentType,"text/html");
else if(strcmp(ext,".gif")==0)sprintf(rs.contentType,"image/gif");
else if(strcmp(ext,".jpg")==0)sprintf(rs.contentType,"image/jpg");
else if(strcmp(ext,".jpeg")==0)sprintf(rs.contentType,"image/jpeg");
else if(strcmp(ext,".png")==0)sprintf(rs.contentType,"image/png");
else if(strcmp(ext,".ico")==0)sprintf(rs.contentType,"image/ico");
else if(strcmp(ext,".zip")==0)sprintf(rs.contentType,"image/zip");
else if(strcmp(ext,".gz")==0)sprintf(rs.contentType,"image/gz");
else if(strcmp(ext,".js")==0)sprintf(rs.contentType,"text/javascript");
else if(strcmp(ext,".css")==0)sprintf(rs.contentType,"text/css");
else if(strcmp(ext,".svg")==0)sprintf(rs.contentType,"image/svg+xml");
else if(strcmp(ext,".woff")==0)sprintf(rs.contentType,"font/woff");
else if(strcmp(ext,".woff2")==0)sprintf(rs.contentType,"font/woff2");
else if(strcmp(ext,".tar")==0)sprintf(rs.contentType,"image/tar");
else if(strcmp(ext,".htm")==0)sprintf(rs.contentType,"text/html");
else if(strcmp(ext,".html")==0)sprintf(rs.contentType,"text/html");
else if(strcmp(ext,".php")==0)sprintf(rs.contentType,"text/html");
else if(strcmp(ext,".pdf")==0)sprintf(rs.contentType,"application/pdf");
else if(strcmp(ext,".zip")==0)sprintf(rs.contentType,"image/zip");
else if(strcmp(ext,".rar")==0)sprintf(rs.contentType,"application/octet-stream");
else if(strcmp(ext,".0")==0)sprintf(rs.contentType,"0");
else sprintf(rs.contentType,"text/html");
rs.httpCode=200;
sendHeader(rs);
k=0;
c = fgetc(file); 
while (c!=EOF) 
{ 
content[k]=c;
c = fgetc(file);
pout(rs,content);
} 
content[k]='\0';
pout(rs,content);
}
}
else
{
rs.httpCode=200;
sprintf(rs.contentType,"text/html");
sendHeader(rs);
pout(rs,"<html><body><H1>404 Error</H1></body></html>");
} 
close(rs.clientSocket);  
}
void postHandler(struct Request r,struct Response rs)
{
int len=strlen(r.url);
if(len>4&&r.url[len-1]=='s'&&r.url[len-2]=='s'&&r.url[len-3]=='.')
{
usersFunction u=requestProcessor(r.url);
u(r,rs);
close(rs.clientSocket);
}
else
{
sendResponse(r,rs);
}
}
void getHandler(struct Request r,struct Response rs)
{
int len=strlen(r.url);
if(len>4&&r.url[len-1]=='s'&&r.url[len-2]=='s'&&r.url[len-3]=='.')
{
usersFunction u=requestProcessor(r.url);
u(r,rs);
close(rs.clientSocket);
}
else
{
sendResponse(r,rs);
}
}
void acceptRequest(int clientSocket)
{
char buffer[10000];
int bufferSize=10000;
int i=0,j=0,k=0;
char url[1000];
char siteName[1000];
struct Pair *pair,*p;
struct Request rq;
struct Response rs;
char cookieName[50];
char cookieValue[50];
char attribute[100];
char value[1000];
char *ext;
struct Cookie *cc,*c;
rq.pair=NULL;
rq.cookie=NULL;
rs.cookie=NULL;
recv(clientSocket, buffer, bufferSize, 0); 
if(strlen(buffer)==0)
{
close(clientSocket);
return;
}
url[0]='\0';
for(i=0;buffer[i]!='/';i++);
i++;
for(j=0;buffer[i]!='/'&&buffer[i]!=' '&&buffer[i]!='\0';j++,i++)siteName[j]=buffer[i];
siteName[j]='\0';
if(buffer[i]==' ')strcpy(url,"index.html");
if(buffer[i]=='/')
{
i++;
if(buffer[i]==' ')strcpy(url,"index.html");
else
{
for(j=0;buffer[i]!=' '&&buffer[i]!='\0'&&buffer[i]!='?';i++,j++)url[j]=buffer[i];
url[j]='\0';
}
}
rs.clientSocket=clientSocket;
strcpy(rq.url,url);
strcpy(rq.siteName,siteName);
if(buffer[i]=='?')
{
pair=NULL;
i++;
while(buffer[i]!=' ')
{
for(j=0;buffer[i]!='=';j++,i++)attribute[j]=buffer[i];
attribute[j]='\0';
i++;
for(j=0;buffer[i]!='&'&&buffer[i]!=' ';j++,i++)value[j]=buffer[i];
value[j]='\0';
p=(struct Pair *)malloc(sizeof(struct Pair));
strcpy(p->attribute,attribute);
strcpy(p->value,value);
p->next=NULL;
if(pair==NULL)
{
pair=p;
rq.pair=p;
}
else
{
pair->next=p;
pair=pair->next;
}
if(buffer[i]=='&')i++;
k++;
}
}
cc=NULL;
for(k=0;buffer[i+7]!='\0'&&k==0;i++)
{
if(buffer[i]=='C'&&buffer[i+1]=='o'&&buffer[i+2]=='o'&&buffer[i+3]=='k'&&buffer[i+4]=='i'&&buffer[i+5]=='e'&&buffer[i+6]==':')
{
i=i+8;
while(1)
{
k=1;
for(j=0;buffer[i]!='=';i++,j++)
{
cookieName[j]=buffer[i];
}
cookieName[j]='\0';
i++;
for(j=0;buffer[i]!=' '&&buffer[i]!=';'&&buffer[i]!='\n';j++,i++)
{
cookieValue[j]=buffer[i];
}
cookieValue[j]='\0';
c=(struct Cookie *)malloc(sizeof(struct Cookie));
strcpy(c->cookieName,cookieName);
strcpy(c->cookieValue,cookieValue);
strcpy(c->cookieExpiry,"");
c->next=NULL;
if(cc==NULL)
{
rq.cookie=c;
cc=c;
}
else
{
cc->next=c;
cc=cc->next;
}
if(buffer[i]=='\n')break;
i+=2;
}
}
}
if(buffer[0]=='G')getHandler(rq,rs);
if(buffer[0]=='P')postHandler(rq,rs);
}
int createClientSocket(struct Socket socket)
{
int serverSocket=socket.serverSocket;
int clientSocket;
socklen_t addrlen; 
if (listen(socket.serverSocket, 100) < 0) {    
perror("server: listen");    
exit(1);    
}
if ((clientSocket = accept(serverSocket, (struct sockaddr *) &(socket.address), &addrlen)) <= 0) {    
return 0;
}    
return clientSocket;     
}
int bindSocket(struct Socket socket)
{
int serverSocket=socket.serverSocket;
int bind_socket=1;
socket.address.sin_family = AF_INET;    
socket.address.sin_addr.s_addr = INADDR_ANY;    
socket.address.sin_port = htons(PORT);    
if ((bind_socket=bind(serverSocket, (struct sockaddr *) &(socket.address), sizeof(socket.address))) != 0){    
return bind_socket;
}
return bind_socket;
}
int createServerSocket()
{
int server_socket;
if ((server_socket = socket(AF_INET, SOCK_STREAM, 0)) <= 0) 
{
return 0;
}
return server_socket;
}  

void firstFunction(struct Request r,struct Response rs)
{
printf("User ka dusra function chla\n");
strcpy(r.url,"index.html");
sendResponse(r,rs);
}
void secondFunction(struct Request r,struct Response rs)
{
int num=0;
char name[26];
char mobileNo[26];
double score=0.0;
char cookie1[25];
char sessionId[25];
cookie1[0]='\0';
mobileNo[0]='\0';
sessionId[0]='\0';
cookie1[0]='\0';
get(r,"%{name}s %{num}d %{mobile}s %{score}f ",name,&num,mobileNo,&score);
printf("Name:%s MobileNo:%s Salary:%d score:%f\n",name,mobileNo,num,score);
setContentType(&rs,"text/html");
setHttpCode(&rs,200);
getCookie(&r,"dob",cookie1);
printf("Dob:-%s\n",cookie1);
setCookie(&rs,"plan","54258","Wed, 21 Oct 2020 07:28:00 GMT ");
setCookie(&rs,"password","12345","Wed, 21 Oct 2020 07:28:00 GMT ");
setCookie(&rs,"dob","17082000","Wed, 21 Oct 2020 07:28:00 GMT ");
getSession(&r,sessionId);
printf("Session Id:-%s\n",sessionId);
setSession(&rs,"hjsdka778hbhj");
sendHeader(rs);
pout(rs,"<html><body><p1>Success</p1>");
pout(rs,"<p>Hurray</p></body></html>");
}
usersFunction mappingFunction(char * url)
{
if(strcmp(url,"first.ss")==0)return secondFunction;
if(strcmp(url,"second.ss")==0)return firstFunction;
return NULL;
}
int main() 
{   
struct sockaddr_in address;
int clientSocket,server_socket,bind_socket;
struct Socket socket;
getcwd(cwd, sizeof(cwd));
populateDS(); 
mapper(mappingFunction);
server_socket=createServerSocket(); 
socket.serverSocket=server_socket;
if(server_socket==0){
printf("The socket was unable to create\n");  
return 0;
}
printf("The socket created and Listening on %d\n",PORT);  
bind_socket=bindSocket(socket);
if(bind_socket!=0)
{
printf("The socket was created but unable to bind on Port:%d\n",PORT);  
return 0;
}
while(1)
{
clientSocket=createClientSocket(socket);
if(clientSocket==0)
{
continue;
}
pthread_t thread_id; 
pthread_create(&thread_id, NULL,acceptRequest,clientSocket);
pthread_join(NULL);  
}
close(server_socket);    
return 0;    
}
```

