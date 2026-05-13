# Contribuindo com n8n Templates

Obrigado pelo interesse em contribuir! 🎉

## Como contribuir

### Reportar problemas
- Use as [Issues](../../issues) para reportar bugs ou sugerir melhorias
- Descreva o problema com detalhes: versão do n8n, mensagem de erro, comportamento esperado

### Enviar um novo template
1. Faça um fork do repositório
2. Crie uma branch: `git checkout -b feat/nome-do-template`
3. Crie uma pasta com nome descritivo em kebab-case
4. Adicione o `workflow.json` (sem credenciais reais)
5. Adicione um `README.md` seguindo o padrão dos outros templates
6. Abra um Pull Request descrevendo o caso de uso

### Padrão de qualidade para templates
- [ ] JSON exportado do n8n sem credenciais reais (IDs substituídos por `YOUR_CREDENTIAL_ID`)
- [ ] README com descrição, integrações, pré-requisitos e instruções de importação
- [ ] Nome de pasta em kebab-case e descritivo
- [ ] Workflow testado e funcional

## Padrão de commits

```
feat: adiciona template de agendamento médico
fix: corrige nó de extração de PDF no template de NF
docs: atualiza README do sistema de delivery
```

## Código de conduta

Seja respeitoso e construtivo. Feedbacks são sempre bem-vindos.
