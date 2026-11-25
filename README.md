# Laboratório Docker — SMTP com Mailpit

Ambiente de e-mail com Docker: um servidor SMTP (Mailpit) e um container cliente
numa rede própria com IPs fixos, para enviar e-mail na mão e capturar o tráfego.

## Topologia

| Container       | Imagem            | IP            | Portas                      |
| --------------- | ----------------- | ------------- | --------------------------- |
| `mailpit`       | `axllent/mailpit` | `172.30.10.2` | `1025` (SMTP), `8025` (web) |
| `cliente_email` | `alpine`          | `172.30.10.3` | —                           |

Rede: `email_network` (bridge), sub-rede `172.30.10.0/24`.

## Como subir

```bash
docker compose up -d
docker ps
```

## Como testar (envio SMTP manual)

```bash
docker exec -it cliente_email sh
apk add --no-cache netcat-openbsd
nc -C mailpit 1025
```

```
EHLO cliente.local
MAIL FROM:<usuario1@teste.com>
RCPT TO:<usuario2@teste.com>
DATA
Subject: Teste SMTP
From: <usuario1@teste.com>
To: <usuario2@teste.com>

Olá! Mensagem de teste.
.
QUIT
```

## Interface web

http://localhost:8025

## Captura de tráfego

```bash
# terminal 1 (antes de enviar o e-mail)
docker exec -it mailpit sh
apk add --no-cache tcpdump
tcpdump -i eth0 -U -w /tmp/captura.pcap port 1025
# Ctrl+C depois do envio

# no host
docker cp mailpit:/tmp/captura.pcap ./captura.pcap
```

No Wireshark: botão direito → **Decode As...** → TCP port `1025` → **SMTP**.

## Cálculo da sub-rede

| Campo          | Valor                         |
| -------------- | ----------------------------- |
| Rede           | 172.30.10.0                   |
| Máscara        | 255.255.255.0 (/24)           |
| Broadcast      | 172.30.10.255                 |
| Gateway        | 172.30.10.1                   |
| Hosts úteis    | 172.30.10.1 até 172.30.10.254 |
| Total de hosts | 254                           |

## Parar

```bash
docker compose down
```
