# Projeto de Redes — CIn-UFPE (Packet Tracer)

Reestruturação da Rede dos Laboratórios do CIn-UFPE. Documentação completa em:
https://app.notion.com/p/Projeto-Packet-Tracer-29040775816c81159cb6d9da99d8968c

## Grupo 13 — Bloco IP `172.20.13.0/24`

Integrantes (nome \<usuário CIn\>):
- Amanda Trinity Gomes Nascimento \<atgn\>
- Luiz Miguel Freitas da Silva \<lmfs3\>
- Maria Luísa Brandão Amaral \<mlba\>
- Mirella Laura Fontinelle Martins \<mlfm\>
- Willian Neves Rupert Jones \<wnrj\>
- Maria Eduarda Torres da Costa Lira \<metcl\>

## Prazos oficiais (Google Classroom — prof. Miguel Gomes de Oliveira)

Cada entrega vale 100 pontos no Classroom (2,0 no total do projeto, que vale 10,0). Anexar sempre **relatório acumulativo + arquivo .pkt**.

| Entrega | Tópico | Data |
|---|---|---|
| E1 | Design Físico e Topologia | **04/09/2026** |
| E2 | Endereçamento VLSM | **16/09/2026** |
| E3 | Segmentação (VLANs/DHCP) | **25/09/2026** |
| E4 | Roteamento Inter-VLAN e Conectividade | **07/10/2026** |
| E5 | Entrega Final (demonstração/desafios) | não detalhada no Notion |

> Nota: as datas dentro das páginas individuais E1-E4 do Notion batem com as do Classroom (fonte oficial). O documento-mãe do Notion tinha datas antigas de outro ano (01/05, 15/05, 29/05, 12/06) — **não usar essas**.

## Arquivo de trabalho

`one.pkt` está em `~/Downloads/one.pkt` — topologia inicial que a Amanda começou. Havia uma cópia duplicada (`one (1).pkt`, idêntica, do navegador) que pode ser apagada.

## Cenário e requisitos gerais

- Bloco `/24` do grupo deve virar 5 sub-redes via **VLSM** (evitar desperdício de IP), para os 5 laboratórios simulados:
  - Lab 1: 35 hosts (2 switches de acesso)
  - Lab 2: 30 hosts (2 switches de acesso)
  - Lab 3: 20 hosts
  - Lab 4: 20 hosts
  - Lab 5: 15 hosts
- Roteador R-CIN conecta à RNP em `2.2.2.2/30` (topologia geral) — mas na prática da E4 o destino simulado da RNP é `200.160.7.186/32` via interface Loopback0 no próprio roteador + rota padrão.

### Equipamentos por grupo

| Equipamento | Modelo | Qtd | Função |
|---|---|---|---|
| Roteador (R-CIN) | 2911 | 1 | Roteamento Inter-VLAN (Router-on-a-Stick) + WAN + DHCP |
| Switch Core | 3560-24PS | 1 | Distribuição/trunks, "Core colapsado" (Layer 3) |
| Switch de Acesso | 2960-24TT | 7 | Acesso dos hosts (Lab 1 e 2 usam 2 switches cada, os outros 1 cada) |
| PCs | — | 120 no total (35+30+20+20+15) | Testes DHCP/conectividade |

## Resumo técnico de cada entrega

### E1 — Design Físico e Topologia
- Modelo hierárquico Cisco (Acesso / Distribuição / Core), aqui com **Core colapsado** no switch 3560.
- Relatório: tabela de equipamentos (modelo, qtd, função, conexões) + print da topologia final no Packet Tracer.
- Critério principal: **racionalidade** — design lógico, organizado, suporta as 5 VLANs.

### E2 — Endereçamento VLSM
- Bloco base: `172.20.13.0/24`.
- Fórmulas:
  - Hosts utilizáveis por sub-rede: `2^H − 2` (H = bits de host)
  - Regra do gateway: `2^H ≥ PCs + 3` (PCs + gateway + rede/broadcast)
  - Máscara CIDR: `32 − H`
- Alocar sub-redes em ordem decrescente de tamanho (maior → menor) para evitar fragmentação.
- Relatório: tabela completa de VLSM (rede, máscara/CIDR, faixa de IPs usáveis, gateway) + análise de eficiência/desperdício do bloco.
- Critério principal: **minimização do desperdício**.

**Endereçamento definido (relatório em [E2 - redes(grupo-13).docx](E2%20-%20redes%28grupo-13%29.docx)):**

| Laboratório | VLAN | Necessidade | Alocados | Rede | CIDR | Faixa de IPs Usáveis | Gateway |
|---|---|---|---|---|---|---|---|
| 1 (A+B) | 10 | 35 hosts | 62 | 172.20.13.0 | /26 | 172.20.13.1 – .62 | 172.20.13.1 |
| 2 (A+B) | 20 | 30 hosts | 62 | 172.20.13.64 | /26 | 172.20.13.65 – .126 | 172.20.13.65 |
| 3 | 30 | 20 hosts | 30 | 172.20.13.128 | /27 | 172.20.13.129 – .158 | 172.20.13.129 |
| 4 | 40 | 20 hosts | 30 | 172.20.13.160 | /27 | 172.20.13.161 – .190 | 172.20.13.161 |
| 5 | 50 | 15 hosts | 30 | 172.20.13.192 | /27 | 172.20.13.193 – .222 | 172.20.13.193 |

Sobra livre: `172.20.13.224/27` (32 endereços, reservado para expansão). Taxa de utilização do bloco: 224/256 = 87,5%.

### E3 — Segmentação (VLANs/DHCP)
- Mapeamento: Lab1→VLAN10, Lab2→VLAN20, Lab3→VLAN30, Lab4→VLAN40, Lab5→VLAN50.
- Primeiro passo obrigatório: ativar a porta do roteador (`no shutdown` na interface física, ex. GigabitEthernet0/0).
- Comandos principais: `vlan <id>` / `name`, `switchport trunk encapsulation dot1q` + `switchport mode trunk` (trunk no switch core, Layer 3), `switchport mode trunk` direto nos switches de acesso (Layer 2, já usa dot1q por padrão), `switchport mode access` + `switchport access vlan <id>` nas portas de host.
- No roteador: sub-interfaces ROAS (`interface Gi0/0.<vlanID>`, `encapsulation dot1Q <vlanID>`, `ip address <gateway> <máscara>`), depois DHCP (`ip dhcp excluded-address <gateway>`, `ip dhcp pool <nome>`, `network <rede> <máscara>`, `default-router <gateway>`, `dns-server 8.8.8.8`).
- Portas trunk sugeridas: Switch-Core Fa0/1-5 (→ switches de acesso) + Gi0/1 (→ roteador); cada switch de acesso usa Gi0/1 (→ core).
- Verificação: `show vlan brief`, `show interfaces trunk`, `show ip dhcp pool`.
- Teste: em cada PC, IP Configuration → DHCP → deve aparecer "DHCP request successful."
- Critério principal: **lógica e funcionalidade** (DHCP funcional + trunks suportando todas as VLANs).

### E4 — Roteamento Inter-VLAN e Conectividade
- Simular a RNP: `interface Loopback0`, `ip address 200.160.7.186 255.255.255.255`, `description` explicando o propósito.
- Rota padrão: `ip route 0.0.0.0 0.0.0.0 <interface Loopback0>`.
- 7 testes obrigatórios (com print no relatório):
  1. DHCP funcionando em cada lab
  2. Ping intra-VLAN (mesmo lab)
  3. Ping para o gateway
  4. Ping inter-VLAN (ex: Lab1 → Lab2) — **crucial**
  5. Ping externo até `200.160.7.186`
  6. `tracert` até um PC de outra VLAN
  7. `tracert` até `200.160.7.186`
- Diagnóstico útil: `show ip interface brief`, `show ip route`, `show interfaces trunk`.
- Observação normal: primeiro ping/tracert pode falhar por causa do ARP, os seguintes devem funcionar.
- Critério principal: **conectividade total** (LAN até o backbone).

## Referências (Notion)

Lista extensa de docs oficiais Cisco e tutoriais (VLSM, VLAN, trunk vs access, Router-on-a-Stick, DHCP, traceroute) na página "Referências Bibliográficas" do Notion — ver link no topo deste arquivo.
