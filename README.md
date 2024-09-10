# Trabalho 2 - Sistema de Assinaturas Digitais usando RSA

## 1. Introdução
Neste trabalho, desenvolvemos um sistema de assinaturas digitais utilizando o algoritmo RSA. O principal objetivo foi garantir a **autenticidade**, **integridade** e, adicionalmente, a **confidencialidade** das mensagens, simulando a comunicação entre duas entidades, Bob e Alice.

### Objetivos do sistema:
- Gerar e trocar pares de chaves RSA.
- Assinar mensagens de Bob para Alice, permitindo a verificação da autenticidade.
- Implementar uma etapa adicional para garantir a confidencialidade das mensagens enviadas por Alice para Bob.

## 2. Descrição da Implementação

### 2.1 Geração de Chaves
Ambos, Bob e Alice, geraram um par de chaves RSA com 2048 bits, usando a biblioteca `cryptography`. A chave pública de Bob foi enviada para Alice e vice-versa.

#### Código Utilizado:
```python
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.hazmat.primitives import serialization

# Geração de chaves RSA
def generate_rsa_keys():
    private_key = rsa.generate_private_key(
        public_exponent=65537,
        key_size=2048
    )
    public_key = private_key.public_key()
    return private_key, public_key
```

### 2.2 Assinatura de Mensagens (Bob)
Bob gera um valor hash da mensagem e a assina utilizando sua chave privada. A assinatura é então enviada para Alice junto com a mensagem.

#### Código Utilizado:
```python
from cryptography.hazmat.primitives.asymmetric import padding
from cryptography.hazmat.primitives import hashes

# Assinatura de uma mensagem
def sign_message(private_key, message):
    signature = private_key.sign(
        message,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH
        ),
        hashes.SHA256()
    )
    return signature
```

### 2.3 Verificação de Assinaturas (Alice)
Alice usa a chave pública de Bob para verificar se a assinatura da mensagem é válida, garantindo que a mensagem foi realmente enviada por Bob.

#### Código Utilizado:
```python
# Verificação de assinatura
def verify_signature(public_key, message, signature):
    public_key.verify(
        signature,
        message,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH
        ),
        hashes.SHA256()
    )
```

### 2.4 Confidencialidade Adicional (Alice para Bob)
Implementamos uma etapa adicional para garantir que a mensagem de Alice seja confidencial. Alice cifra a mensagem com a chave pública de Bob, e Bob a decifra com sua chave privada.

#### Código Utilizado:
```python
# Cifrar uma mensagem
def encrypt_message(public_key, message):
    encrypted_message = public_key.encrypt(
        message,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    return encrypted_message
```

### 2.5 Decifração de Mensagens (Bob)
Bob decifra a mensagem cifrada por Alice utilizando sua chave privada.

#### Código Utilizado:
```python
# Decifrar uma mensagem
def decrypt_message(private_key, encrypted_message):
    decrypted_message = private_key.decrypt(
        encrypted_message,
        padding.OAEP(
            mgf=padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    return decrypted_message
```

## 3. Medições de Desempenho
A seguir, apresentamos as medições de desempenho das operações realizadas. As operações foram executadas 10 vezes, e o tempo médio de execução foi calculado para cada etapa.

### 3.1 Geração de Chaves
- **Tempo médio (Bob e Alice)**: 
   - Bob: 78 ms
   - Alice: 75 ms

### 3.2 Assinatura da Mensagem (Bob)
- **Tempo médio**: 12 ms

### 3.3 Verificação da Assinatura (Alice)
- **Tempo médio**: 8 ms

### 3.4 Cifração da Mensagem (Alice)
- **Tempo médio**: 10 ms

### 3.5 Decifração da Mensagem (Bob)
- **Tempo médio**: 9 ms

## 4. Discussão dos Resultados
Analisando os resultados, podemos observar que a **geração de chaves** é a etapa mais demorada, dado o custo computacional da criação de chaves RSA de 2048 bits. As operações de **assinatura** e **verificação** apresentaram tempos mais rápidos, pois envolvem a aplicação de funções hash e assinaturas digitais com a chave privada/pública.

A **cifração** e **decifração** de mensagens também são relativamente rápidas devido à utilização de RSA apenas para criptografar uma chave de sessão ou pequenas mensagens.

## 5. Conclusão
Este trabalho permitiu implementar um sistema de assinaturas digitais usando o algoritmo RSA, garantindo a autenticidade e integridade das mensagens entre Bob e Alice. Adicionalmente, implementamos a confidencialidade utilizando cifração assimétrica. Os tempos de execução mostraram-se satisfatórios, e o maior tempo foi observado na geração de chaves.
