# Projeto Linux AWS: Infraestrutura Web com Monitoramento Automatizado



## 🌐 VPC Personalizada

Criei uma nova VPC personalizada com as seguintes configurações:

* CIDR Block: 10.0.0.0/16
* Sub-rede pública
* Tabela de rotas com acesso à internet (gateway de internet criado e associado)
* Segurança configurada via grupo de segurança (SG)


---

## 🖥 EC2 – Instância Linux

Criei uma instância EC2 Amazon Linux 2023, configurada da seguinte forma:

* Associada à VPC personalizada
* IP elástico atribuído (fixo para acessar o site publicamente)
* Grupo de segurança liberando a porta 80 (HTTP)
* Sistema operacional: Amazon Linux 2023 (free tier)

### 🔐 Conexão

* Acesso via terminal com chave .pem (SSH)
* Acesso habilitado via AWS Systems Manager (SSM), eliminando a necessidade de abrir porta 22

---

## 🌐 Instalação do NGINX

Acesse a instância via terminal e execute:

```bash
sudo yum update -y
sudo amazon-linux-extras enable nginx1
sudo yum install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
``` 




### 💻 Código HTML do site de boas-vindas:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <title>Projeto Linux</title>
  <style>
    body {
      background-color: #f0f8ff;
      color: #333;
      font-family: Arial, sans-serif;
      text-align: center;
      padding-top: 100px;
    }
    h1 {
      font-size: 48px;
      color: #2e8b57;
    }
  </style>
</head>
<body>
  <h1>🚀 Bem-vindo ao meu Projeto Linux!</h1>
</body>
</html>
```

---

## 🔐 IAM – Segurança e Permissões

Inicialmente a instância EC2 estava com status "não gerenciado" (Managed: false) no Systems Manager.

### ✅ Correções aplicadas:

* Criei uma *IAM Role* com a política AmazonSSMManagedInstanceCore
* Associei essa role à EC2

Assim ficou possível acessar com *Session Manager* sem expor portas ou usar chaves SSH.

---

## 📈 Monitoramento do Site + Alertas no Telegram

Criei o arquivo monitoramento.sh com o seguinte conteúdo:

```bash

#!/bin/bash
source /home/ec2-user/monitoramento/.env

SITE="http://3.132.135.34"
STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$SITE")

MENSAGEM="🚨 O site está fora do ar! Status: $STATUS - $(date)"

if [ "$STATUS" != "200" ]; then
    echo "$(date): O site está fora do ar! Status: $STATUS" >> /home/ec2-user/site_offline.log
    curl -s -X POST "https://api.telegram.org/bot$BOT_TOKEN/sendMessage" \
         -d chat_id="$CHAT_ID" \
         -d text="$MENSAGEM"
fi
```

---

## ⏰ Automação com Crontab

Adicionei o script no crontab -e para rodar a cada 1 minuto:

```bash
* * * * * /bin/bash /home/ec2-user/monitoramento/monitoramento.sh

```
---

## 🖼 AMI – Imagem Personalizada do Projeto

Criei uma AMI privada a partir da instância já configurada permitindo assim recriar ou replicar uma infraestrutura rapidamente, com toda a configuração pronta.

* NGINX instalado e em execução
* Página HTML publicada
* Scripts de monitoramento e cron configurados
* Acesso seguro com IAM Role e SSM

### 🔧 Como foi criada:

1. Acesse: EC2 > Imagens > AMIs
2. Clique em "*Criar imagem*" a partir da instância configurada
3. Nome da AMI: monitoramento-vpc-nova
4. Visibilidade: *Privada*
5. ID da AMI gerado: ami-0d0eacf79b739391a



---

## RESULTADO FINAL ✅

### 🌐  site: http://3.132.135.34 

Imagens da fucionalidade do site + monitoramento 
![funcionalidadedosite](https://github.com/user-attachments/assets/cb6ed793-0bbf-4aaa-8586-a117f7934740)
![funcionalidadedomonitoramento](https://github.com/user-attachments/assets/7f2669c0-92fe-49f5-872c-132a86f09c73)



