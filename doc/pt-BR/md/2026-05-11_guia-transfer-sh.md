# Guia de Uso — transfer.firebee.com.br

## Visão Geral

O **transfer.sh** é um serviço de compartilhamento de arquivos leve, auto-hospedado, acessado inteiramente via linha de comando com `curl`. Não há interface web de upload, não há conta para criar — basta fazer o upload de um arquivo e compartilhar a URL gerada.

A instância Firebee está disponível em:

```
http://transfer.firebee.com.br:3210
```

> **Importante:** a porta `:3210` é obrigatória em todas as URLs. O serviço não responde na porta padrão 80/443.

---

## Upload de Arquivos

### Sintaxe básica

```bash
curl --upload-file <caminho-do-arquivo> http://transfer.firebee.com.br:3210
```

O servidor retorna uma URL única para o arquivo. Exemplo real:

```bash
$ curl --upload-file ./teste.txt http://transfer.firebee.com.br:3210
http://transfer.firebee.com.br:3210/G2k6d6aMSl/teste.txt
```

A URL gerada contém um identificador aleatório (`G2k6d6aMSl`) seguido do nome original do arquivo.

### Exemplos por tipo de arquivo

**Arquivo de texto:**
```bash
curl --upload-file ./relatorio.txt http://transfer.firebee.com.br:3210
```

**Arquivo comprimido:**
```bash
curl --upload-file ./backup.tar.gz http://transfer.firebee.com.br:3210
```

**Imagem:**
```bash
curl --upload-file ./screenshot.png http://transfer.firebee.com.br:3210
```

**Qualquer extensão funciona** — o nome do arquivo na URL preserva a extensão original.

---

## Download de Arquivos

Para baixar, use a URL retornada no upload diretamente no navegador ou via `curl`:

```bash
curl http://transfer.firebee.com.br:3210/G2k6d6aMSl/teste.txt -o teste.txt
```

Ou salve com o nome original automaticamente:

```bash
curl -O http://transfer.firebee.com.br:3210/G2k6d6aMSl/teste.txt
```

---

## Upload via Pipe (stdin)

É possível enviar dados diretamente de um pipe, sem criar arquivo intermediário:

```bash
cat arquivo.txt | curl -X PUT --upload-file "-" http://transfer.firebee.com.br:3210/arquivo.txt
```

**Casos de uso com pipe:**

Comprimir e enviar em um único comando:
```bash
tar czf - ./pasta/ | curl -X PUT --upload-file "-" http://transfer.firebee.com.br:3210/pasta.tar.gz
```

Enviar saída de um comando:
```bash
ps aux | curl -X PUT --upload-file "-" http://transfer.firebee.com.br:3210/processos.txt
```

Dump de banco de dados direto para o serviço:
```bash
mongodump --archive | curl -X PUT --upload-file "-" http://transfer.firebee.com.br:3210/backup.archive
```

---

## Alias / Função de Conveniência

Adicione ao `~/.bashrc` ou `~/.zshrc` para facilitar o uso:

```bash
transfer() {
  if [ -z "$1" ]; then
    echo "Uso: transfer <arquivo>"
    return 1
  fi
  curl --upload-file "$1" "http://transfer.firebee.com.br:3210/$(basename "$1")"
  echo  # nova linha após a URL
}
```

Após recarregar o shell (`source ~/.bashrc`), o uso fica simples:

```bash
transfer relatorio.pdf
transfer backup.tar.gz
```

---

## Compartilhando com Outras Pessoas

A URL retornada pelo upload pode ser compartilhada com qualquer pessoa que tenha acesso à rede onde o serviço está exposto. Basta enviar a URL — não é necessária autenticação para baixar.

```
http://transfer.firebee.com.br:3210/G2k6d6aMSl/teste.txt
                                    ^^^^^^^^^
                                    ID único gerado automaticamente
```

> **Atenção de segurança:** a URL é o único "segredo" do arquivo. Qualquer pessoa que tiver a URL pode baixá-lo. Não compartilhe URLs de arquivos sensíveis em canais públicos.

---

## Limites do Serviço

| Parâmetro | Valor |
|---|---|
| Tamanho máximo por arquivo | **5 GB** |
| Tempo de retenção | **14 dias** (arquivos são deletados automaticamente) |
| Rate limit | **10 requisições** por janela de tempo |
| Provedor de armazenamento | Local (disco do servidor VPS) |

Arquivos são apagados automaticamente após 14 dias. Se precisar de persistência maior, faça novo upload antes do vencimento.

---

## Resolução de Problemas

### A URL não abre no navegador
Verifique se a porta `:3210` está incluída na URL. O serviço não responde em `http://transfer.firebee.com.br` sem a porta.

### Erro de rate limit
Se receber erro HTTP 429, aguarde alguns instantes e tente novamente. O limite é de 10 requisições por janela.

### Upload lento ou interrompido
Para arquivos grandes, use a flag `--limit-rate` para controlar a velocidade e evitar timeouts:
```bash
curl --upload-file ./arquivo-grande.tar.gz --limit-rate 10M http://transfer.firebee.com.br:3210
```

### Verificar progresso do upload
```bash
curl --upload-file ./arquivo.tar.gz http://transfer.firebee.com.br:3210 --progress-bar
```

---

## Configuração do Serviço (Referência)

O serviço é executado via Docker Compose na VPS:

```yaml
image: dutchcoders/transfer.sh:latest
porta: 3210 → 8080 (interna)
armazenamento: volume local /data
max upload: 5 GB
purge: 14 dias
```

Repositório de configuração: `transfer-sh-firebee/docker-compose.yaml`
