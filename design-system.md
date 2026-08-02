# Design System - Pombo Correio

Este documento detalha o sistema de design visual do **Pombo Correio**, projetado com uma estética **Premium Dark Mode & Glassmorphism**. Ele serve como guia para que desenvolvedores e outras IAs mantenham a consistência visual em novos módulos e telas da aplicação.

Os tokens e classes utilitárias correspondentes estão implementados em [design-system.css](file:///Users/vinicius/doc-pombocorreio/design-system.css).

---

## 1. Tokens de Cores (Paleta HSL/Hex)

| Token CSS | Cor | Uso Principal |
| :--- | :--- | :--- |
| `--bg-color` | `#0b0f19` | Fundo principal da aplicação. |
| `--bg-darker` | `#070a13` | Fundos de sidebar e overlays mais opacos. |
| `--card-bg` | `rgba(22, 28, 45, 0.6)` | Fundo translúcido para painéis e cartões. |
| `--card-border` | `rgba(255, 255, 255, 0.08)` | Bordas finas de containers translúcidos. |
| `--accent-color` | `#3b82f6` | Azul neon para destaques, links e botões primários. |
| `--success-color` | `#10b981` | Verde esmeralda para status "Válido", "Ativo" ou ações de sucesso. |
| `--warning-color` | `#f59e0b` | Amarelo âmbar para alertas, inatividades de tempo (30d). |
| `--danger-color` | `#ef4444` | Vermelho carmesim para status "Inválido", "Inativo" ou ações destrutivas. |
| `--purple-color` | `#8b5cf6` | Roxo para bandeiras manuais e destaques de campanhas. |

---

## 2. Tipografia

A fonte primária é a **Inter** (carregada via Google Fonts).

* **Tamanhos (`font-size`)**:
  * `--text-xs` (0.75rem / 12px) - Legendas, tags e metadados.
  * `--text-sm` (0.875rem / 14px) - Texto corrido, conteúdo de tabelas.
  * `--text-base` (1rem / 16px) - Descrições e labels de formulários.
  * `--text-lg` (1.125rem / 18px) - Subtítulos e botões.
  * `--text-xl` (1.25rem / 20px) - Títulos de cartões.
  * `--text-2xl` (1.5rem / 24px) - Títulos de cabeçalhos de página e modais.
* **Pesos (`font-weight`)**:
  * `300` (Light)
  * `400` (Normal)
  * `500` (Medium)
  * `600` (Semibold)
  * `700` (Bold)

---

## 3. Estrutura de Layout e Grid

### Sidebar Lateral (Navegação)
Use `<aside class="app-sidebar">` com a largura fixa de `260px` fixada à esquerda.

### Conteúdo Principal
Use `<main class="app-main">` com um recuo à esquerda de `260px` para não sobrepor a sidebar, e espaçamento interno flexível.

---

## 4. Componentes Clave

### Painéis Translúcidos (Glassmorphism)
```html
<div class="glass-panel">
    <!-- Conteúdo do Painel -->
</div>
```
*Adicione `.glass-panel-interactive` se quiser efeito de hover/flutuação ao passar o mouse.*

### Tabelas de Dados
A classe `.data-table` estiliza tabelas sem bordas pesadas e com cabeçalho sutil. Para linhas clicáveis, utilize a classe `.row-interactive`:
```html
<div class="table-container">
    <table class="data-table">
        <thead>
            <tr>
                <th>Item</th>
                <th>Status</th>
            </tr>
        </thead>
        <tbody>
            <tr class="row-interactive">
                <td>Nome do Item</td>
                <td><span class="badge badge-success">Ativo</span></td>
            </tr>
        </tbody>
    </table>
</div>
```

### Botões e Formulários
* Botões usam `.btn` combinado com variantes de cores: `.btn-primary`, `.btn-secondary`, `.btn-danger`.
* Inputs, selectores e áreas de texto usam `.form-control`.
* Agrupamentos de formulário usam `.form-group` com label em maiúsculas usando `.form-label`.

### Painel Detalhado Deslizante (Slide-over Drawer)
Utilizado para fichas completas sem recarregar a tela:
* `.detail-drawer` com classe `.open` para torná-lo visível.
* `.overlay-backdrop` com classe `.active` para escurecer e borrar o fundo.

---

## 5. Exemplo de Uso Prático para Outras IAs

Para criar uma nova página (ex: **Campanhas**) seguindo este padrão, instrua a IA com o seguinte prompt:

> *"Crie uma página de gestão de Campanhas utilizando o Design System do Pombo Correio. Importe a folha de estilos [design-system.css](file:///Users/vinicius/doc-pombocorreio/design-system.css). Monte a estrutura com sidebar à esquerda e cabeçalho sutil. O corpo deve conter um `.glass-panel` para os filtros, uma `.table-container` com `.data-table` contendo a lista de campanhas (Nome, Gatilho, Ações, Status) e um `.detail-drawer` que desliza da direita ao clicar na linha, revelando a lista de clientes participantes da campanha e a timeline de envios executados."*
