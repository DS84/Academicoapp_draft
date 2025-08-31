# Upload de Documentos - Academicoapp

Sistema de upload de documentos com tema laranja, branco, preto e cinzento do Academicoapp.

## 📁 Estrutura de Arquivos

```
projeto/
├── upload.html          # Página principal
├── css/
│   └── upload.css       # Estilos personalizados
├── js/
│   └── upload.js        # Funcionalidades JavaScript
└── README.md           # Documentação
```

## 🎨 Tema de Cores

- **Laranja Primário**: `#ff6b35`
- **Laranja Secundário**: `#ff8c42`
- **Laranja Claro**: `#ffab73`
- **Cinza Escuro**: `#2c3e50`
- **Cinza Médio**: `#6c757d`
- **Cinza Claro**: `#f8f9fa`
- **Branco**: `#ffffff`
- **Preto**: `#000000`

## ✨ Funcionalidades

### 📤 Upload Avançado
- **Drag & Drop**: Arraste arquivos diretamente
- **Validação**: Tipos permitidos (PDF, DOC, DOCX, PPT, PPTX)
- **Limite de Tamanho**: Máximo 10MB
- **Barra de Progresso**: Feedback visual do upload
- **Preview**: Mostra informações do arquivo selecionado

### 🎯 Validação em Tempo Real
- Validação de campos obrigatórios
- Feedback visual (bordas verdes/vermelhas)
- Alertas personalizados
- Mensagens de erro específicas

### 📱 Design Responsivo
- Layout adaptável para mobile
- Animações suaves
- Formas flutuantes decorativas
- Glass morphism effect

## 🚀 Como Usar

### 1. Estrutura de Pastas
Crie a estrutura de pastas conforme mostrado acima.

### 2. Dependências Externas
O projeto usa CDNs externos (já incluídos no HTML):
- Bootstrap 5.3.0
- Material Icons (Google Fonts)

### 3. Integração com Backend

Para integrar com um backend real, modifique a função `sendToBackend()` no arquivo `js/upload.js`:

```javascript
function sendToBackend(formData) {
    const formDataObj = new FormData();
    
    // Adicionar todos os campos
    formDataObj.append('title', formData.title);
    formDataObj.append('description', formData.description);
    formDataObj.append('category', formData.category);
    formDataObj.append('level', formData.level);
    formDataObj.append('allowDownload', formData.allowDownload);
    formDataObj.append('file', formData.file);
    
    // Enviar para seu endpoint
    fetch('/seu-endpoint-aqui', {
        method: 'POST',
        body: formDataObj
    })
    .then(response => response.json())
    .then(data => {
        if (data.success) {
            showSuccessMessage();
        } else {
            showAlert(data.message, 'error');
        }
    })
    .catch(error => {
        showAlert('Erro de conexão', 'error');
    });
}
```

### 4. Personalização

#### Alterar Limite de Arquivo
No `upload.js`, linha ~147:
```javascript
const maxSize = 10 * 1024 * 1024; // Altere aqui (em bytes)
```

#### Adicionar Tipos de Arquivo
No `upload.js`, linha ~139:
```javascript
const validTypes = ['pdf', 'doc', 'docx', 'ppt', 'pptx', 'txt']; // Adicione aqui
```

#### Modificar Redirecionamento
No `upload.js`, função `submitDocument()`:
```javascript
window.location.href = "sua-pagina.html"; // Altere aqui
```

## 🔧 Funcionalidades JavaScript

### Principais Funções
- `initializeUpload()`: Inicializa todas as funcionalidades
- `handleFile(file)`: Processa arquivo selecionado
- `validateFormData(data)`: Valida dados do formulário
- `submitDocument(formData)`: Envia documento
- `showAlert(message, type)`: Mostra alertas personalizados

### Eventos Suportados
- Drag and drop de arquivos
- Validação em tempo real
- Upload com barra de progresso
- Alertas automáticos
- Redirecionamento após sucesso

## 📋 Campos do Formulário

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| Título | Text | Sim | Título do documento |
| Descrição | Textarea | Não | Descrição opcional |
| Arquivo | File | Sim | Arquivo para upload |
| Categoria | Select | Sim | Categoria do documento |
| Nível | Select | Não | Nível educacional |
| Download | Checkbox | Não | Permitir download |

## 🎭 Categorias Disponíveis
- 📚 E-book
- 📄 Artigo  
- 📋 Apostila
- 🎓 TCC
- 🎥 Vídeo

## 📚 Níveis Educacionais
- 📝 Ensino Fundamental
- 🎒 Ensino Médio
- 🎓 Ensino Superior
- 👨‍🎓 Pós-Graduação

## 🐛 Debugging

Para debug, use as funções disponíveis no console:
```javascript
// Ver dados atuais do formulário
debugFormData();

// Resetar formulário completamente
resetForm();
```

## 📱 Compatibilidade
- Chrome 80+
- Firefox 75+
- Safari 13+
- Edge 80+
- Dispositivos móveis (iOS/Android)

## 💡 Notas Importantes

1. **Segurança**: Sempre valide arquivos no backend
2. **Performance**: Considere upload em chunks para arquivos grandes
3. **UX**: Mantenha feedback visual durante operações
4. **Acessibilidade**: Todos os elementos têm suporte a screen readers

---

**Desenvolvido para Academicoapp** 🎓