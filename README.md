# 🌐 Análise de Incidentes de Rede: Investigação de DNS & SYN Flood

## 🎯 Objetivo do Projeto
Este projeto documenta a investigação técnica de uma interrupção de serviço em uma aplicação web real (`www.yummyrecipesforme.com`). O foco foi aplicar análise de pacotes de baixo nível com **tcpdump** e **Wireshark** para diagnosticar falhas de conectividade e identificar ataques de negação de serviço (DoS).

## 📝 Cenário da Atividade 
Como Analista de Segurança em uma agência de publicidade, recebi alertas de que o site institucional estava fora do ar. 
- **Sintoma do Usuário:** Erro de "Pausa na conexão" (Connection Timeout).
- **Missão:** Determinar se a falha era uma configuração incorreta de rede ou um ataque cibernético ativo.

---

## 🛠️ Análise Técnica 

### 🔍 Incidente 1: Falha Crítica de Resolução DNS
Utilizando o `tcpdump`, analisei a tentativa de resolução de nome do host local (`192.51.100.15`) para o servidor DNS (`203.0.113.2`).
- **Protocolo:** UDP na Porta 53.
- **Evidência de Log:** O servidor retornou pacotes **ICMP Tipo 3 (Destination Unreachable)** com a mensagem `"udp port 53 unreachable"`.
- **Diagnóstico:** O serviço de DNS estava inativo ou bloqueado, impedindo que o navegador obtivesse o endereço IP necessário para carregar o site.

### 🌪️ Incidente 2: Ataque SYN Flood (Negação de Serviço)
Através da inspeção de logs do **Wireshark**, identifiquei um padrão clássico de exploração do protocolo TCP:
- **Padrão Detectado:** Uma inundação massiva de pacotes com a flag **[SYN]** ativa, sem o correspondente envio do pacote **ACK** final pelo cliente.
- **Exploração técnica:** O atacante forçou o servidor a manter milhares de "conexões meio-abertas", exaurindo as tabelas de recursos (backlog) e impedindo o acesso de usuários legítimos.
- **Impacto:** Negação de serviço (DoS) por esgotamento de memória/CPU.

---
### 🔎 Evidência da Captura (Wireshark)
| Time | Source IP | Destination IP | Protocol | Flags | Info |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 13:24:32.19 | 192.51.100.15 | 203.0.113.2 | TCP | [SYN] | Port 80, Seq 0, Win 64240 |
| 13:24:32.45 | 192.51.100.15 | 203.0.113.2 | TCP | [SYN] | Port 80, Seq 0, Win 64240 |
| 13:24:32.72 | 192.51.100.15 | 203.0.113.2 | TCP | [SYN] | Port 80, Seq 0, Win 64240 |
| 13:24:32.98 | 192.51.100.15 | 203.0.113.2 | TCP | [SYN] | Port 80, Seq 0, Win 64240 |

> **Nota:** O padrão acima demonstra a inundação de pacotes de sincronização (SYN) sem a resposta final (ACK), esgotando os recursos do servidor.

## ✅ Resposta e Plano de Ação 

Como parte da resposta ao incidente, as seguintes medidas de mitigação foram propostas:

1.  **Bloqueio de Perímetro:** Implementação imediata de regras de ACL no Firewall para descartar (drop) pacotes originados dos endereços IP identificados na inundação.
2.  **Hardening do Stack TCP:** Recomendação de ativação de **SYN Cookies** no kernel do sistema operacional, permitindo que o servidor gerencie conexões legítimas mesmo sob ataque.
3.  **Configuração de Rate Limiting:** Ajuste de limites de requisições por segundo em Firewalls de Próxima Geração (NGFW) para detectar e barrar anomalias de tráfego antes que atinjam o servidor web.
4.  **Resiliência de DNS:** Implementação de redundância com servidores DNS secundários para evitar pontos únicos de falha na resolução de nomes.

---

## 🎓 Conclusão
Este projeto demonstra que a segurança eficaz nasce da compreensão profunda dos protocolos de rede. Unindo minha formação em **Análise e Desenvolvimento de Sistemas (ADS)** com técnicas de **Defesa Cibernética**, pude não apenas identificar o "que" estava acontecendo, mas o "como" a arquitetura do protocolo estava sendo explorada.

---
**Ferramentas Utilizadas:**
* `tcpdump` (Linha de comando para captura rápida)
* `Wireshark` (Análise detalhada de fluxo TCP/HTTP)
* Protocolos: TCP, UDP, ICMP, DNS
