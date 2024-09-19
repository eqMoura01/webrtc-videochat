# Configuração HTTPS:
1. npm install mkcert -g
2. mkcert create-ca
3. mkcert create-cert
4. OPCIONAL: para rodar localmente, atualize os arquivos com seu IP local

# Passos em um aplicativo WebRTC
1. getUserMedia() é executado - CLIENTE1/Init/Chamada/Ofertante
2. CLIENTE1 cria um objeto RTCPeerConnection chamado peerConnection
3. o novo peerConnection precisa de servidores STUN para que outros clientes possam encontrá-lo
    - eles criarão candidatos ICE, para uso futuro
4. CLIENTE1 adiciona os tracks do localstream (do GUM) ao peerConnection
    - precisamos associar o feed do CLIENTE1 com o peerConnection
5. CLIENTE1 cria uma oferta
    - a oferta requer um peerConnection com tracks
    - oferta = RTCSessionDescription, um objeto com 2 propriedades
        1. SDP - codec e outras informações
        2. Tipo (oferta)
6. CLIENTE1 passa a oferta para peerConnection.setLocalDescription
7. (ASSÍNCRONO) Candidatos ICE podem agora começar a chegar

O servidor de sinalização precisa estar em execução para 8 em diante
- o servidor de sinalização é um servidor node, ele permite que os navegadores se encontrem e se comuniquem

8. CLIENTE1 emite a oferta para o servidor de sinalização (socket.io/node)
    - o servidor socket.io a mantém para o outro navegador
    - associar com CLIENTE1
9. (ASSÍNCRONO) À medida que o item 7 ocorre, emita candidatos ICE para o servidor de sinalização
    - o servidor socket.io a mantém para quando outro cliente responder
    - associar os candidatos ICE com CLIENTE1

## CLIENTE1 e Servidor de Sinalização aguardam CLIENTE2

10. CLIENTE2 carrega a página da web
    - io.connect() é executado e conecta-se ao servidor socket.io
11. socket.io emite a RTCSessionDescription para o novo cliente
    - RTCSessionDescription = uma oferta
12. CLIENTE2 executa getUserMedia()
13. CLIENTE2 cria um peerConnection
    - passa os servidores STUN
14. CLIENTE2 adiciona seus tracks do localstream ao peerConnection
15. CLIENTE2 cria uma resposta com createAnswer()
    - createAnswer = RTCSessionDescription
    - igual ao item #5, mas com um tipo de "resposta"
16. CLIENTE2 entrega a resposta para peerConnection.setLocalDescription()
17. CLIENTE2 possui a oferta (SDP do CLIENTE1)
    - CLIENTE2 pode passar a oferta para peerConnection.setRemoteDescription()
18. (ASSÍNCRONO) Uma vez que o item #16 é executado, CLIENTE2 pode começar a coletar candidatos ICE

## O servidor de sinalização (socket.io) tem esperado...

19. CLIENTE2 emite a resposta (RTCSessionDesc - sdp/tipo) para o servidor de sinalização
20. (ASSÍNCRONO) CLIENTE2 irá ouvir por tracks/ICE do remoto.
    - e está pronto.
    - aguardando candidatos ICE do CLIENTE1
    - aguardando tracks do CLIENTE1
21. O servidor de sinalização tem ouvido pela resposta. Ao chegar
    - emitir a resposta do CLIENTE2 para o CLIENTE1 (RTCSessionDesc - sdp/tipo)
22. CLIENTE1 recebe a resposta e a passa para pc.setRemoteDescription()
23. (ASSÍNCRONO) CLIENTE1 aguarda candidatos ICE e tracks

## 21 & 23 estão aguardando candidatos ICE.
    - Uma vez que eles sejam trocados, os tracks serão trocados
# CONECTADO!!