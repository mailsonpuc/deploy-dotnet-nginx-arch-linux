## /etc/nginx/nginx.conf 

```bash
#user http;
worker_processes 1;

#error_log logs/error.log;
#error_log logs/error.log notice;
#error_log logs/error.log info;

#pid logs/nginx.pid;


# ============================================================
# CARREGA TODOS OS MÓDULOS INSTALADOS DO NGINX
# ============================================================
include modules.d/*.conf;


# ============================================================
# CONFIGURAÇÃO DE EVENTOS
# ============================================================
events {
    worker_connections 1024;
}


# ============================================================
# CONFIGURAÇÃO HTTP
# ============================================================
http {

    # ========================================================
    # TIPOS MIME
    #
    # Necessário para o React/Vite funcionar corretamente.
    #
    # Sem essa configuração:
    # .js  pode ser enviado como application/octet-stream
    # .css pode ser enviado como application/octet-stream
    #
    # O Firefox então bloqueia esses arquivos.
    # ========================================================
    include /etc/nginx/mime.types;

    # Tipo padrão para arquivos que não possuem MIME definido.
    default_type application/octet-stream;


    # ========================================================
    # LOG
    # ========================================================

    #log_format main '$remote_addr - $remote_user [$time_local] "$request" '
    #                '$status $body_bytes_sent "$http_referer" '
    #                '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log logs/access.log main;


    # ========================================================
    # PERFORMANCE
    # ========================================================

    # Permite que o Nginx envie arquivos diretamente.
    sendfile on;

    #tcp_nopush on;

    #keepalive_timeout 0;

    # Mantém conexões HTTP abertas por até 65 segundos.
    keepalive_timeout 65;

    #gzip on;


    # ========================================================
    # SERVIDOR
    # ========================================================
    server {

        # Nginx ficará escutando na porta 80.
        listen 80;

        # Nome do servidor.
        server_name localhost;


        # ====================================================
        # FRONTEND REACT / VITE
        # ====================================================

        # Pasta gerada pelo:
        #
        # yarn build
        #
        # Dentro dessa pasta temos:
        #
        # dist/
        # ├── index.html
        # ├── assets/
        # ├── favicon.svg
        # └── ...
        #
        root /home/mailson/petShoopFullStack/src/frontend/PetShoopFront/dist;

        # Arquivo inicial do frontend.
        index index.html;


        # ====================================================
        # ROTAS DO FRONTEND REACT
        # ====================================================

        location / {

            # Primeiro tenta encontrar o arquivo solicitado.
            #
            # Exemplo:
            # /favicon.svg
            # /assets/index.js
            # /assets/index.css
            #
            # Se não encontrar, envia para index.html.
            #
            # Isso é importante para React Router.
            try_files $uri $uri/ /index.html;
        }


        # ====================================================
        # API .NET
        # ====================================================
        #
        # Todas as requisições que começarem com:
        #
        # /api/
        #
        # serão encaminhadas para:
        #
        # http://localhost:5130
        #
        # Exemplo:
        #
        # http://localhost/api/v1/Clientes
        #
        # será encaminhado para:
        #
        # http://localhost:5130/api/v1/Clientes
        #
        # ====================================================

        location /api/ {

            # Redireciona a requisição para a API .NET.
            proxy_pass http://localhost:5130;


            # =================================================
            # CONFIGURAÇÃO DO PROXY
            # =================================================

            # Usa HTTP/1.1 entre Nginx e API.
            proxy_http_version 1.1;


            # Mantém suporte para Upgrade de conexão.
            proxy_set_header Upgrade $http_upgrade;

            # Mantém a conexão.
            proxy_set_header Connection keep-alive;


            # Envia o Host original para a API.
            proxy_set_header Host $host;


            # Envia o IP real do cliente.
            proxy_set_header X-Real-IP $remote_addr;


            # Mantém a cadeia de IPs.
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;


            # Informa para a API qual protocolo foi utilizado.
            proxy_set_header X-Forwarded-Proto $scheme;


            # Evita problemas de cache durante Upgrade.
            proxy_cache_bypass $http_upgrade;
        }


        # ====================================================
        # PÁGINAS DE ERRO
        # ====================================================

        # Caso a API ou Nginx tenha erro 500, 502, 503 ou 504,
        # será apresentada a página 50x.html.
        error_page 500 502 503 504 /50x.html;


        # Localização da página de erro.
        location = /50x.html {

            root /usr/share/nginx/html;
        }


        # ====================================================
        # CONFIGURAÇÃO PHP
        # ====================================================
        #
        # Não utilizada neste projeto.
        #
        # O código abaixo fica apenas como referência.
        # ====================================================

        #location ~ \.php$ {
        #    proxy_pass http://127.0.0.1;
        #}


        # ====================================================
        # FASTCGI
        # ====================================================
        #
        # Não utilizado pelo React/.NET.
        # ====================================================

        #location ~ \.php$ {
        #    root html;
        #    fastcgi_pass 127.0.0.1:9000;
        #    fastcgi_index index.php;
        #    fastcgi_param SCRIPT_FILENAME /scripts$fastcgi_script_name;
        #    include fastcgi_params;
        #}


        # ====================================================
        # BLOQUEIO DE .HTACCESS
        # ====================================================
        #
        # Não é necessário para React/.NET, mas fica como
        # referência.
        # ====================================================

        #location ~ /\.ht {
        #    deny all;
        #}
    }


    # ========================================================
    # OUTRO VIRTUAL HOST
    # ========================================================
    #
    # Exemplo de outro servidor rodando em outra porta.
    #
    #server {
    #    listen 8000;
    #    listen somename:8080;
    #    server_name somename alias another.alias;

    #    location / {
    #        root html;
    #        index index.html index.htm;
    #    }
    #}


    # ========================================================
    # HTTPS
    # ========================================================
    #
    # Configuração de HTTPS para uso futuro.
    #
    # Para utilizar HTTPS será necessário possuir um certificado.
    #
    #server {
    #    listen 443 ssl;
    #    server_name localhost;

    #    ssl_certificate cert.pem;
    #    ssl_certificate_key cert.key;

    #    ssl_session_cache shared:SSL:1m;
    #    ssl_session_timeout 5m;

    #    ssl_ciphers HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers on;

    #    location / {
    #        root html;
    #        index index.html index.htm;
    #    }
    #}

}

```

---

## /etc/systemd/system/petshoop-api.service

```bash

# --- SEÇÃO DE METADADOS E DEPENDÊNCIAS ---
[Unit]
# Descrição amigável do serviço exibida nos logs e comandos do systemd
Description=PetShoop API .NET

# Define que este serviço só deve iniciar após a rede do sistema estar operacional
After=network.target

# --- SEÇÃO DE CONFIGURAÇÕES DE EXECUÇÃO ---
[Service]
# Diretório de trabalho onde o serviço será executado (pasta de publicação do .NET)
WorkingDirectory=/home/mailson/petShoopFullStack/src/backend/PetShoop.API/bin/Publish

# Comando exato para iniciar o executável da sua API
ExecStart=/home/mailson/petShoopFullStack/src/backend/PetShoop.API/bin/Publish/PetShoop.API

# Configura o serviço para reiniciar automaticamente em qualquer falha ou fechamento
Restart=always

# Tempo de espera (em segundos) antes de tentar reiniciar a API em caso de queda
RestartSec=10

# Define o ambiente da aplicação ASP.NET Core como Produção
Environment=ASPNETCORE_ENVIRONMENT=Production

# Define a porta local em que a API vai escutar requisições (http://localhost:5000)
Environment=ASPNETCORE_URLS=http://localhost:5000

# Variáveis para indicar o caminho exato da instalação do SDK/Runtime .NET no sistema
Environment="DOTNET_ROOT=/home/mailson/dotnet"
Environment="DOTNET_ROOT_X64=/home/mailson/dotnet"

# Atualiza a variável PATH para o systemd reconhecer executáveis do .NET na pasta do usuário
Environment="PATH=/home/mailson/dotnet:/home/mailson/.dotnet/tools:/usr/local/sbin:/usr/local/bin:/usr/bin"

# Variável de ambiente com a string de conexão do banco de dados SQL Server
Environment="ConnectionStrings__DefaultConnection=Server=127.0.0.1;Database=clearArchitecture;User=sa;Password=1q2w3e4r@#$;TrustServerCertificate=True;"

# Variável de ambiente com a chave secreta usada para assinar/validar tokens JWT
Environment="Jwt__SecretKey=SVOM^]Nw%Ns{,I7rH"

# Usuário do Linux com permissão para executar o processo do serviço
User=mailson

# --- SEÇÃO DE INICIALIZAÇÃO NO BOOT ---
[Install]
# Define que o serviço deve ser iniciado no modo multi-usuário (padrão de inicialização do servidor)
WantedBy=multi-user.target

```
