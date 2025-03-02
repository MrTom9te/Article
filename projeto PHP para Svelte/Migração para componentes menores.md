segue abaixo uma lista dos componentes que devem ser criados 

1. **Componentes de Layout**
```
Layout/
├── Header.svelte
├── MainLayout.svelte
└── MobileHeader.svelte
```

2. **Componentes de Navegação**
```
Navigation/
├── MainMenu.svelte
├── MenuItem.svelte
└── SearchBar.svelte
```

1. **Componentes de Post**
```
Post/
├── PostBox.svelte (área de criação de post)
├── PostList.svelte (container de posts)
├── PostCard.svelte (card individual de post)
├── PostActions.svelte (botões de ação do post)
└── PostAttachments.svelte (área de anexos)
```

2. **Componentes de Sidebar**
```
Sidebar/
├── TrendingSection.svelte
├── TrendingTopic.svelte
├── FollowSuggestions.svelte
└── FollowCard.svelte
```

3. **Componentes UI Comuns**
```
Common/
├── Avatar.svelte
├── Button.svelte
├── Icon.svelte
└── SearchInput.svelte
```

4. **Stores (Estado)**
```
stores/
├── userStore.js
├── postsStore.js
└── trendingStore.js
```

5. **Estrutura de Arquivos Sugerida**
```
src/
├── lib/
│   ├── components/
│   │   ├── Layout/
│   │   ├── Navigation/
│   │   ├── Post/
│   │   ├── Sidebar/
│   │   └── Common/
│   ├── stores/
│   ├── services/
│   │   ├── api.js
│   │   └── auth.js
│   └── utils/
├── routes/
└── app.html
```

6. **Principais Features por Componente**

- **MainLayout.svelte**
  - Grid principal do app
  - Gerenciamento de layout responsivo

- **Header.svelte**
  - Logo
  - Menu de navegação
  - Perfil do usuário

- **PostBox.svelte**
  - Input para novo post
  - Upload de mídia
  - Ações de postagem

- **PostCard.svelte**
  - Informações do usuário
  - Conteúdo do post
  - Interações (curtir, comentar, compartilhar)

- **TrendingSection.svelte**
  - Lista de trending topics
  - Sugestões de quem seguir
  - Barra de pesquisa

7. **Considerações para Migração**

- Implementar sistema de rotas
- Gerenciamento de estado com stores
- Sistema de autenticação
- Comunicação com API
- Tratamento de responsividade
- Gestão de assets e imagens
- Temas e estilização global

8. **Próximos Passos**

9. Criar estrutura base do projeto Svelte
10. Migrar componentes um por um
11. Implementar sistema de estado
12. Criar serviços de API
13. Testar responsividade
14. Implementar autenticação

