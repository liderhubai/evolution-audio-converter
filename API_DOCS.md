# Evolution Audio/Video/Image Converter API

Esta API oferece conversão de arquivos de áudio, vídeo e imagem usando FFmpeg.

## Endpoints Disponíveis

### 1. `/process-audio` (POST)
Converte arquivos de áudio para diferentes formatos.

**Parâmetros:**
- `file`: Arquivo de áudio (multipart/form-data)
- `base64`: Dados em base64 do arquivo
- `url`: URL do arquivo para download
- `format`: Formato de saída (padrão: "ogg")
  - Formatos suportados: ogg, mp3, wav, m4a, mp4
- `transcribe`: "true" para incluir transcrição (padrão: "false")
- `language`: Idioma para transcrição (opcional)

**Resposta:**
```json
{
  "duration": 120,
  "format": "ogg",
  "audio": "base64_encoded_data", // ou "url" se S3 estiver habilitado
  "transcription": "texto transcrito" // se solicitado
}
```

### 2. `/process-video` (POST)
Converte arquivos de vídeo para diferentes formatos.

**Parâmetros:**
- `file`: Arquivo de vídeo (multipart/form-data)
- `base64`: Dados em base64 do arquivo
- `url`: URL do arquivo para download
- `format`: Formato de saída (padrão: "mp4")
  - Formatos suportados: mp4, webm, avi

**Resposta:**
```json
{
  "duration": 300,
  "format": "mp4",
  "video": "base64_encoded_data" // ou "url" se S3 estiver habilitado
}
```

### 3. `/process-image` (POST)
Converte arquivos de imagem para diferentes formatos.

**Parâmetros:**
- `file`: Arquivo de imagem (multipart/form-data)
- `base64`: Dados em base64 do arquivo
- `url`: URL do arquivo para download
- `format`: Formato de saída (padrão: "jpg")
  - Formatos suportados: jpg, jpeg, png, webp, gif

**Resposta:**
```json
{
  "format": "jpg",
  "image": "base64_encoded_data" // ou "url" se S3 estiver habilitado
}
```

### 4. `/transcribe` (POST)
Transcreve apenas o áudio sem conversão.

**Parâmetros:**
- `file`: Arquivo de áudio (multipart/form-data)
- `base64`: Dados em base64 do arquivo
- `url`: URL do arquivo para download
- `language`: Idioma para transcrição (opcional)

**Resposta:**
```json
{
  "transcription": "texto transcrito"
}
```

## Autenticação

Todas as rotas requerem autenticação via header `apikey`:

```
apikey: sua_chave_aqui
```

## Configurações de Ambiente

```env
# Chave da API
API_KEY=sua_chave_secreta

# CORS
CORS_ALLOW_ORIGINS=https://meusite.com,https://outro.com

# Transcrição (opcional)
ENABLE_TRANSCRIPTION=true
TRANSCRIPTION_PROVIDER=openai # ou groq
OPENAI_API_KEY=sk-...
GROQ_API_KEY=gsk_...
TRANSCRIPTION_LANGUAGE=pt

# S3 Storage (opcional)
ENABLE_S3_STORAGE=true
S3_ENDPOINT=s3.amazonaws.com
S3_ACCESS_KEY=...
S3_SECRET_KEY=...
S3_BUCKET_NAME=meu-bucket
S3_REGION=us-east-1
S3_USE_SSL=true
S3_URL_EXPIRATION=24h
```

## Exemplos de Uso

### Converter vídeo para MP4
```bash
curl -X POST http://localhost:8080/process-video \
  -H "apikey: sua_chave" \
  -F "file=@video.avi" \
  -F "format=mp4"
```

### Converter imagem para WebP
```bash
curl -X POST http://localhost:8080/process-image \
  -H "apikey: sua_chave" \
  -F "file=@imagem.png" \
  -F "format=webp"
```

### Converter áudio com transcrição
```bash
curl -X POST http://localhost:8080/process-audio \
  -H "apikey: sua_chave" \
  -F "file=@audio.mp3" \
  -F "format=ogg" \
  -F "transcribe=true" \
  -F "language=pt"
```

## Dependências

- FFmpeg deve estar instalado no sistema
- Go 1.19+
- Dependências do Go especificadas em `go.mod`

## Formatos Suportados

### Vídeo
- **MP4**: H.264 + AAC, otimizado para web
- **WebM**: VP9 + Opus, para navegadores modernos
- **AVI**: H.264 + MP3, compatibilidade ampla

### Áudio
- **OGG**: Opus codec, alta qualidade/baixo tamanho
- **MP3**: Compatibilidade universal
- **WAV**: Audio sem compressão
- **M4A**: AAC em container MP4
- **MP4**: Para áudio extraído de vídeo

### Imagem
- **JPG/JPEG**: Compressão com perda, ótimo para fotos
- **PNG**: Sem perda, suporte à transparência
- **WebP**: Formato moderno, menor tamanho
- **GIF**: Animações e transparência simples 