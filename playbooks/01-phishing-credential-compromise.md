# Playbook — Phishing com Roubo de Credencial

**Autor:** Alessandro Sendim  
**Atualizado em:** Março/2026  
**Severidade:** Alta  
**Referências:** NIST SP 800-61 · MITRE ATT&CK  

---

## O que é esse cenário?

Phishing é quando o atacante manda um e-mail falso pra induzir o usuário a entregar a senha dele numa página que imita um serviço real — Microsoft 365, VPN, portal interno. O perigo não é só perder a senha: é o que o atacante faz depois que entra. Movimentação lateral, acesso a arquivos, criação de backdoors. Se demorar pra detectar, um phishing simples vira um incidente crítico.

---

## Como esse incidente chega até o SOC?

- Usuário reporta e-mail suspeito
- SIEM alerta login de localização fora do padrão
- Gateway de e-mail bloqueia URL maliciosa
- Usuário diz que "não consegue mais logar na conta"

> Se o usuário não consegue mais logar, provavelmente o atacante já trocou a senha. Trate como comprometimento confirmado.

---

## Antes de agir — perguntas de triagem

```
[ ] O usuário clicou no link ou só recebeu o e-mail?
[ ] Ele inseriu a senha na página falsa?
[ ] O acesso malicioso ainda está ativo agora?
[ ] Essa conta tem permissões de admin?
[ ] Outros usuários receberam o mesmo e-mail?
```

| Situação | Severidade |
|----------|------------|
| E-mail recebido, link não clicado | Média — bloquear domínio |
| Link clicado, senha não inserida | Média/Alta — investigar |
| Senha inserida + acesso ativo | Crítico — agir imediatamente |
| Conta admin comprometida | Crítico — escalar |

---

## Resposta ao incidente

### 1. Identificação

Analisar o e-mail malicioso:
- Verificar o remetente real (o campo "De" pode estar mascarado)
- Checar o domínio: tem typosquatting? (`micros0ft.com`, `bradesco-suporte.net`)
- Pegar as URLs do e-mail e verificar no VirusTotal **sem clicar**
- Se tiver anexo, checar o hash no VirusTotal também

Verificar logs de autenticação:
- Login fora do horário normal do usuário?
- Login de outro país ou IP desconhecido?
- Após o login, acessou arquivos ou criou regras de encaminhamento de e-mail?

---

### 2. Contenção

Executar nessa ordem:

```
1. Resetar a senha da conta comprometida
   (não avisar o usuário antes — o atacante pode estar lendo os e-mails)

2. Revogar todas as sessões ativas
   (trocar a senha não basta — tokens de sessão ainda dão acesso)

3. Bloquear o domínio malicioso no gateway de e-mail e no firewall

4. Se suspeitar de malware na máquina, isolar da rede
```

---

### 3. Erradicação

```
[ ] Remover regras de encaminhamento de e-mail criadas pelo atacante
[ ] Verificar se foram criados novos usuários ou alteradas permissões
[ ] Verificar apps OAuth autorizados pela conta durante o período
[ ] Verificar se houve download em massa de arquivos
```

> Regras de encaminhamento maliciosas são a persistência mais esquecida na resposta. Sempre checar via linha de comando, não só pela interface gráfica.

---

### 4. Recuperação

```
1. Reativar a conta após confirmar que está limpa
2. Forçar recadastro de MFA (evitar SMS — preferir app autenticador)
3. Conversar com o usuário sobre o que aconteceu, sem culpar
4. Monitorar a conta por 30 dias após o incidente
```

---

### 5. Lições aprendidas

Após fechar o incidente, responder:
- Quanto tempo levou entre o comprometimento e a detecção?
- A detecção foi proativa (SIEM) ou o usuário que reportou?
- Os logs estavam disponíveis e com retenção suficiente?
- O que pode melhorar no processo?

---

## Evidências a preservar

| Evidência | Fonte |
|-----------|-------|
| E-mail original com headers | Cliente de e-mail |
| Screenshot da página de phishing (com URL visível) | Browser |
| Logs de autenticação do período | Azure AD / IdP |
| IP de origem do acesso malicioso | Logs de auth |
| Hash do anexo malicioso | EDR / VirusTotal |

```bash
# Registrar hash das evidências coletadas
sha256sum evidencia_email.eml
sha256sum logs_autenticacao.csv
```

---

## MITRE ATT&CK

| Tática | Técnica | ID |
|--------|---------|-----|
| Initial Access | Phishing: Spearphishing Link | T1566.002 |
| Credential Access | Steal Web Session Cookie | T1539 |
| Persistence | Account Manipulation | T1098.002 |
| Collection | Email Collection | T1114 |

---

*Feito como parte do meu portfólio de estudos em SOC/Blue Team.*  
*[github.com/alesendim](https://github.com/alesendim)*
