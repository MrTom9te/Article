
---

- [x] **1. Configuração Básica do Servidor**

    **Título:** Iniciando um Servidor Simples

    **Exercício:** Crie um servidor Actix Web que escute na porta 8080 e responda à rota raiz ("/") com a mensagem "Servidor Funcionando!".

    **Requisitos:**

    *   O servidor deve ser iniciado corretamente.
    *   A resposta deve ser enviada com sucesso.

    **Resposta Esperada:** Ao acessar `http://127.0.0.1:8080/` no navegador, você deve ver a mensagem "Servidor Funcionando!".

    **Exemplos Vagos:**

    *   Lembre-se da estrutura básica: `HttpServer::new`, `App::new`, `.route`, `.bind`, `.run`, `.await`.
    *   Use uma função handler assíncrona simples.

---

- [x] **2. Número de Workers**

    **Título:** Definindo o Número de Workers

    **Exercício:** Modifique o exercício anterior para que o servidor utilize explicitamente 2 threads (workers).

    **Requisitos:**

    *   O servidor deve ser configurado para usar 2 workers.
    *   O restante do comportamento deve permanecer o mesmo.

    **Resposta Esperada:** O servidor deve funcionar da mesma forma, mas internamente estará usando 2 threads. Não há uma forma direta de *verificar* isso no navegador, mas você pode usar ferramentas de monitoramento do sistema para observar o número de threads do processo.

    **Exemplos Vagos:**

    *   Use o método `.workers()` do `HttpServer`.

---

- [x] **3. Handler Assíncrono**

    **Título:** Simulando uma Operação Demorada

    **Exercício:** Crie um servidor com uma rota `/demorado` que simule uma operação demorada (por exemplo, 5 segundos) *sem* bloquear a thread do worker.

    **Requisitos:**

    *   Use uma função handler assíncrona.
    *   Use `tokio::time::sleep` para simular a demora.  (Não use `std::thread::sleep`!).
    *   A resposta deve ser "Operação Concluída!".

    **Resposta Esperada:** Ao acessar `/demorado`, o navegador ficará "carregando" por 5 segundos e depois exibirá "Operação Concluída!". Durante esses 5 segundos, o servidor *deve* continuar respondendo a outras requisições (por exemplo, se você tiver a rota `/` do exercício 1, ela deve continuar funcionando).

    **Exemplos Vagos:**

    *   Lembre-se de usar `async fn` e `.await` para a função `sleep`.
    *   Importe `tokio::time::sleep`.

---

- [x] **4. HTTPS com OpenSSL**

    **Título:** Configurando um Servidor Seguro

    **Exercício:** Crie um servidor Actix Web que utilize HTTPS (TLS) com OpenSSL.

    **Requisitos:**

    *   Gere um certificado e uma chave privada usando o comando `openssl` (como mostrado no artigo). *Não é necessário remover a senha da chave para este exercício*.
    *   Configure o `HttpServer` para usar `bind_openssl`.
    *   A rota raiz ("/") deve responder com "Conexão Segura!".

    **Resposta Esperada:** Ao acessar `https://127.0.0.1:8443/` (observe o `https` e a porta `8443`), você deve ver a mensagem "Conexão Segura!". O navegador provavelmente mostrará um aviso de segurança porque o certificado é autoassinado (não é de uma autoridade confiável), mas você pode ignorar o aviso para fins de teste.

    **Exemplos Vagos:**

    *   Você precisará da dependência `openssl` no seu `Cargo.toml`.
    *   Use `SslAcceptor::mozilla_intermediate`, `set_private_key_file`, e `set_certificate_chain_file`.
    *   Use uma porta diferente de 8080 (ex: 8443) para HTTPS.
    * Crie o certificado e a chave. Exemplo do comando, mas não se esqueça de alterar as informações para algo mais relevante para você:
        ```bash
        openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -sha256 -subj "/C=BR/ST=MeuEstado/L=MinhaCidade/O=MinhaOrganizacao/OU=MeuSetor/CN=localhost"
        ```

---

- [x] **5. Keep-Alive Personalizado**

    **Título:** Ajustando o Tempo de Keep-Alive

    **Exercício:** Crie um servidor que responda na rota `/` e configure o keep-alive para 15 segundos.

    **Requisitos:**

    *   Configure o keep-alive explicitamente.
    *   Use um valor de 15 segundos.

    **Resposta Esperada:**  Não há uma forma *fácil* de verificar isso diretamente no navegador, pois o comportamento de keep-alive é, em grande parte, transparente. No entanto, se você usar ferramentas de desenvolvimento do navegador ou um analisador de tráfego de rede (como o Wireshark), poderá observar que a conexão TCP é mantida aberta por 15 segundos após a resposta ser enviada.

    **Exemplos Vagos:**
    *   Use o método `.keep_alive()` do `HttpServer`.
    * Lembre-se de usar `actix_web::http::KeepAlive::Timeout(15)` ou `Duration::from_secs(15)`.

---

- [x] **6. Shutdown Gracioso**

    **Título:** Configurando o Timeout de Desligamento

    **Exercício:** Modifique um dos exercícios anteriores (por exemplo, o exercício 1) para configurar o timeout de desligamento gracioso para 5 segundos.

    **Requisitos:**

    *   Use o método `shutdown_timeout`.
    *   Defina o valor para 5 segundos.

    **Resposta Esperada:** Novamente, isso não é facilmente observável no navegador. Para testar *adequadamente*, você precisaria:
        1.  Iniciar o servidor.
        2.  Fazer uma requisição que demore mais de 5 segundos para ser processada (você pode usar o exercício 3 como base, mas modifique o tempo para, digamos, 10 segundos).
        3.  Enquanto a requisição ainda estiver sendo processada, pressione Ctrl+C para enviar um sinal de desligamento ao servidor.
        4.  O servidor *deve* esperar até que a requisição demorada seja concluída (ou até que os 5 segundos se esgotem) antes de realmente parar.

    **Exemplos Vagos:**

    *   Use o método `.shutdown_timeout()` do `HttpServer`.

---