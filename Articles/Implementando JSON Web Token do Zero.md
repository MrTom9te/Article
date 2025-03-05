# JWT: Implementando JSON Web Token do Zero

Um JSON Web Token (JWT) é uma forma compacta e segura de transmitir informações entre diferentes partes em formato JSON. Neste artigo, vamos implementar um JWT do zero, sem depender de bibliotecas específicas para JWT, apenas com funcionalidades básicas de codificação e criptografia.

## Entendendo a estrutura do JWT

Antes de começar a implementação, é importante entender que um JWT é composto por três partes separadas por pontos:

```
header.payload.signature
```

1. **Header**: Contém o tipo do token e o algoritmo de assinatura usado
2. **Payload**: Contém as afirmações (claims) sobre uma entidade
3. **Signature**: Garante a integridade do token

Cada parte é codificada em Base64Url, que é similar ao Base64 tradicional, mas seguro para uso em URLs.

## Implementando JWT do zero em Rust

Vamos criar uma implementação simples de JWT usando apenas funções básicas de codificação e criptografia.

### 1. Dependências necessárias

Precisaremos de algumas bibliotecas básicas:

```rust
use base64::{engine::general_purpose, Engine as _};
use hmac::{Hmac, Mac};
use sha2::Sha256;
use serde::{Serialize, Deserialize};
use serde_json::{json, Value};
use std::error::Error;
use std::time::{SystemTime, UNIX_EPOCH};
```

### 2. Estruturas de dados para o JWT

```rust
#[derive(Serialize, Deserialize)]
struct Header {
    alg: String,
    typ: String,
}

#[derive(Serialize, Deserialize)]
struct Claims {
    sub: String,        // Subject (geralmente o ID do usuário)
    name: String,       // Nome do usuário
    role: String,       // Papel/permissão
    exp: u64,           // Tempo de expiração
    iat: u64,           // Tempo de emissão
}
```

### 3. Funções para codificação/decodificação Base64Url

Para implementar o Base64Url corretamente:

```rust
fn base64url_encode(input: &[u8]) -> String {
    general_purpose::URL_SAFE_NO_PAD.encode(input)
}

fn base64url_decode(input: &str) -> Result<Vec<u8>, Box<dyn Error>> {
    Ok(general_purpose::URL_SAFE_NO_PAD.decode(input)?)
}
```

### 4. Criação de JWT do zero

```rust
fn create_jwt(user_id: &str, username: &str, role: &str, secret: &[u8]) -> Result<String, Box<dyn Error>> {
    // Criando o header
    let header = Header {
        alg: "HS256".to_string(),
        typ: "JWT".to_string(),
    };
    
    // Obtendo o timestamp atual
    let now = SystemTime::now()
        .duration_since(UNIX_EPOCH)?
        .as_secs();
    
    // Criando as claims
    let claims = Claims {
        sub: user_id.to_string(),
        name: username.to_string(),
        role: role.to_string(),
        iat: now,
        exp: now + 3600, // Expira em 1 hora
    };
    
    // Serializando para JSON
    let header_json = serde_json::to_string(&header)?;
    let claims_json = serde_json::to_string(&claims)?;
    
    // Codificando em Base64Url
    let encoded_header = base64url_encode(header_json.as_bytes());
    let encoded_payload = base64url_encode(claims_json.as_bytes());
    
    // Concatenando header e payload para assinar
    let signing_input = format!("{}.{}", encoded_header, encoded_payload);
    
    // Criando a assinatura HMAC-SHA256
    let mut mac = Hmac::<Sha256>::new_from_slice(secret)
        .map_err(|_| "HMAC error")?;
    mac.update(signing_input.as_bytes());
    let signature = mac.finalize().into_bytes();
    
    // Codificando a assinatura em Base64Url
    let encoded_signature = base64url_encode(&signature);
    
    // Montando o token final
    let token = format!("{}.{}.{}", encoded_header, encoded_payload, encoded_signature);
    
    Ok(token)
}
```

### 5. Verificação e decodificação de JWT

```rust
fn verify_jwt(token: &str, secret: &[u8]) -> Result<Claims, Box<dyn Error>> {
    // Dividindo o token em suas partes
    let parts: Vec<&str> = token.split('.').collect();
    if parts.len() != 3 {
        return Err("Token inválido: formato incorreto".into());
    }
    
    let encoded_header = parts[0];
    let encoded_payload = parts[1];
    let encoded_signature = parts[2];
    
    // Recriando a assinatura para verificar
    let signing_input = format!("{}.{}", encoded_header, encoded_payload);
    
    // Calculando o HMAC-SHA256
    let mut mac = Hmac::<Sha256>::new_from_slice(secret)
        .map_err(|_| "HMAC error")?;
    mac.update(signing_input.as_bytes());
    let signature = mac.finalize().into_bytes();
    
    // Verificando a assinatura
    let expected_signature = base64url_encode(&signature);
    if encoded_signature != expected_signature {
        return Err("Assinatura inválida".into());
    }
    
    // Decodificando o payload
    let payload_json = base64url_decode(encoded_payload)?;
    let payload_str = String::from_utf8(payload_json)?;
    let claims: Claims = serde_json::from_str(&payload_str)?;
    
    // Verificando a expiração
    let now = SystemTime::now()
        .duration_since(UNIX_EPOCH)?
        .as_secs();
    
    if claims.exp < now {
        return Err("Token expirado".into());
    }
    
    Ok(claims)
}
```

### 6. Exemplo de uso

Aqui está como usar nossas funções em um exemplo prático:

```rust
fn main() -> Result<(), Box<dyn Error>> {
    // Nossa chave secreta
    let secret = b"minha_chave_secreta_e_segura_para_assinatura";
    
    // Criando um token
    println!("Criando token JWT...");
    let token = create_jwt("1234", "Maria Silva", "admin", secret)?;
    println!("Token gerado: {}\n", token);
    
    // Decodificando o token manualmente para demonstração
    let parts: Vec<&str> = token.split('.').collect();
    let header_json = String::from_utf8(base64url_decode(parts[0])?)?;
    let payload_json = String::from_utf8(base64url_decode(parts[1])?)?;
    
    println!("Header decodificado: {}", header_json);
    println!("Payload decodificado: {}\n", payload_json);
    
    // Verificando o token
    println!("Verificando token...");
    match verify_jwt(&token, secret) {
        Ok(claims) => {
            println!("Token válido para usuário: {}", claims.name);
            println!("Papel do usuário: {}", claims.role);
            println!("Válido até: {}", claims.exp);
        },
        Err(e) => println!("Erro ao verificar token: {}", e),
    }
    
    // Exemplo com token inválido
    println!("\nTestando token com assinatura inválida...");
    let invalid_token = format!("{}.{}.ASSINATURA_INVALIDA", parts[0], parts[1]);
    match verify_jwt(&invalid_token, secret) {
        Ok(_) => println!("O token deveria ser inválido!"),
        Err(e) => println!("Erro esperado: {}", e),
    }
    
    Ok(())
}
```

## Entendendo cada etapa da implementação

### Processo de criação do JWT

1. **Criação do header e payload**: Definimos as informações necessárias para o token, como algoritmo, tipo, ID do usuário, nome, etc.

2. **Serialização para JSON**: Convertemos as estruturas para strings JSON.

3. **Codificação Base64Url**: Transformamos as strings JSON em strings seguras para URL.

4. **Criação da assinatura**: Usamos HMAC-SHA256 para criar uma assinatura criptográfica do header e payload.

5. **Montagem do token**: Juntamos as três partes com pontos para formar o token final.

### Processo de verificação do JWT

1. **Separação do token**: Dividimos o token em header, payload e assinatura.

2. **Verificação da assinatura**: Recriamos a assinatura usando a chave secreta e comparamos com a assinatura do token.

3. **Decodificação do payload**: Convertemos de Base64Url para texto e depois para uma estrutura de dados.

4. **Verificação da expiração**: Checamos se o token ainda é válido com base no timestamp atual.

## Exemplo prático: Autenticação em uma API simples

Vamos ver como implementar um fluxo de autenticação básico usando nosso JWT:

```rust
struct User {
    id: String,
    username: String,
    password: String, // Em produção, armazenaria apenas o hash
    role: String,
}

struct ApiRequest {
    path: String,
    headers: std::collections::HashMap<String, String>,
}

fn login_endpoint(username: &str, password: &str, users: &[User]) -> Result<String, &'static str> {
    // Simulando verificação de credenciais
    let user = users.iter().find(|u| u.username == username && u.password == password);
    
    match user {
        Some(user) => {
            // Credenciais válidas, gerar token
            create_jwt(&user.id, &user.username, &user.role, b"minha_chave_secreta")
                .map_err(|_| "Erro ao gerar token")
        },
        None => Err("Credenciais inválidas"),
    }
}

fn protected_endpoint(request: &ApiRequest, secret: &[u8]) -> Result<String, &'static str> {
    // Extrair token do cabeçalho Authorization
    let auth_header = request.headers.get("Authorization")
        .ok_or("Cabeçalho Authorization não encontrado")?;
    
    if !auth_header.starts_with("Bearer ") {
        return Err("Formato inválido do cabeçalho Authorization");
    }
    
    let token = &auth_header[7..]; // Removendo "Bearer "
    
    // Verificar token
    let claims = verify_jwt(token, secret)
        .map_err(|_| "Token inválido ou expirado")?;
    
    // Se chegou aqui, o token é válido
    Ok(format!("Dados protegidos para o usuário: {} ({})", claims.name, claims.role))
}

fn simulate_api() {
    // Dados simulados
    let users = vec![
        User {
            id: "1".to_string(),
            username: "admin".to_string(),
            password: "senha123".to_string(),
            role: "admin".to_string(),
        },
    ];
    
    let secret = b"minha_chave_secreta";
    
    // Simulando login
    println!("Tentando login...");
    let token = match login_endpoint("admin", "senha123", &users) {
        Ok(token) => {
            println!("Login bem-sucedido! Token: {}", token);
            token
        },
        Err(e) => {
            println!("Erro no login: {}", e);
            return;
        }
    };
    
    // Simulando requisição para endpoint protegido
    let mut headers = std::collections::HashMap::new();
    headers.insert("Authorization".to_string(), format!("Bearer {}", token));
    
    let request = ApiRequest {
        path: "/api/protected".to_string(),
        headers,
    };
    
    println!("\nAcessando recurso protegido...");
    match protected_endpoint(&request, secret) {
        Ok(response) => println!("Resposta: {}", response),
        Err(e) => println!("Erro: {}", e),
    }
}
```

## Considerações de segurança

Ao implementar JWT do zero, lembre-se de alguns pontos importantes:

1. **Guarde sua chave secreta com segurança**: Se a chave for comprometida, qualquer pessoa pode gerar tokens válidos.

2. **Sempre defina um tempo de expiração**: Tokens sem expiração representam um risco de segurança.

3. **Use HTTPS**: Sempre transmita JWTs através de conexões seguras.

4. **Não armazene dados sensíveis**: O payload do JWT não é criptografado, apenas codificado em Base64.

5. **Considere implementar uma lista de revogação**: Em sistemas críticos, você pode precisar invalidar tokens antes da expiração.

6. **Atenção com a validação das claims**: Verifique todas as claims relevantes, especialmente a expiração.

## Conclusão

Criar um JWT do zero nos dá uma compreensão profunda de como essa tecnologia funciona. Embora em ambientes de produção seja recomendável usar bibliotecas especializadas e bem testadas, implementar um JWT manualmente é um excelente exercício para entender os conceitos fundamentais de autenticação e segurança em aplicações modernas.

Com o conhecimento adquirido neste artigo, você pode adaptar essa implementação para suas necessidades específicas ou migrar com confiança para uma biblioteca especializada, sabendo exatamente o que acontece nos bastidores.

Lembre-se que a autenticação é apenas uma parte de uma estratégia de segurança abrangente - mas com JWTs implementados corretamente, você tem uma base sólida para construir sistemas seguros e escaláveis.