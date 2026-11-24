# Material de Estudo: Redes para Datacenter

**Objetivo:** Projetar, configurar e diagnosticar redes — a base do datacenter.

Este material consolida os principais conceitos de redes necessários para atuar em ambientes de datacenter. Cada seção traz explicações teóricas, exemplos práticos e orientações de estudo/prática.

---

## 1. Modelos de Referência, Endereçamento e Sub-redes

### 1.1 Modelos OSI e TCP/IP

#### Modelo OSI (Open Systems Interconnection)

O modelo OSI é um framework conceitual de 7 camadas que descreve como os dados viajam de uma aplicação em um host até outra aplicação em outro host.

| Camada | Nome | Função principal | Exemplos de protocolos/tecnologias |
| -------- | ------------------- | ----------------------------------------------- | ----------------------------------------- |
| 7 | Aplicação | Interface com o usuário / serviços de rede | HTTP, HTTPS, FTP, DNS, SMTP, SSH |
| 6 | Apresentação | Formatação, criptografia, compressão | SSL/TLS, JPEG, MPEG |
| 5 | Sessão | Estabelecimento, gerenciamento e encerramento de sessões | NetBIOS, RPC |
| 4 | Transporte | Entrega fim-a-fim, confiabilidade, portas | TCP, UDP |
| 3 | Rede | Roteamento, endereçamento lógico | IP, ICMP, OSPF, BGP |
| 2 | Enlace de Dados | Enquadramento, endereçamento físico (MAC), detecção de erros | Ethernet, PPP, switches |
| 1 | Física | Transmissão de bits no meio físico | Cabos, fibras, sinais elétricos/ópticos |

**Fluxo de dados:**

- **Encapsulamento (descida):** Dados da aplicação → Segmento (Transporte) → Pacote (Rede) → Quadro (Enlace) → Bits (Física).
- **Desencapsulamento (subida):** Processo inverso no destino.

#### Modelo TCP/IP (mais usado na prática)

É um modelo de 4 (ou 5) camadas, mais próximo da implementação real da Internet.

| Camada TCP/IP | Camadas OSI correspondentes | Protocolos principais |
| ------------------- | ----------------------------- | ------------------------------- |
| Aplicação | 5, 6 e 7 | HTTP, DNS, DHCP, FTP, SMTP |
| Transporte | 4 | TCP, UDP |
| Internet | 3 | IP, ICMP, ARP |
| Acesso à Rede | 1 e 2 | Ethernet, Wi-Fi, PPP |

**Comparação importante:**

- O TCP/IP é o modelo **real** usado na Internet.
- O OSI é excelente para **estudo e troubleshooting** (ajuda a isolar problemas por camada).

### 1.2 IPv4 e IPv6

#### IPv4

- Endereço de 32 bits (ex: `192.168.10.25`).
- Escrito em notação decimal pontuada.
- Classes tradicionais (A, B, C) foram substituídas por CIDR.
- Problema: esgotamento de endereços → solução: NAT + IPv6.

**Tipos de endereços IPv4:**

- **Unicast:** um-para-um
- **Broadcast:** um-para-todos na rede
- **Multicast:** um-para-grupo

#### IPv6

- Endereço de 128 bits (ex: `2001:0db8:85a3:0000:0000:8a2e:0370:7334`).
- Notação hexadecimal com compressão de zeros.
- Não usa broadcast (usa multicast e anycast).
- Autoconfiguração (SLAAC) e endereço link-local (`fe80::/10`).

**Principais diferenças:**

| Característica     | IPv4              | IPv6                     |
|--------------------|-------------------|--------------------------|
| Tamanho            | 32 bits           | 128 bits                 |
| Cabeçalho          | Variável (20–60 B)| Fixo (40 B) + extensões  |
| Broadcast          | Sim               | Não                      |
| Segurança          | Opcional (IPsec)  | Obrigatório (IPsec)      |
| Configuração       | Manual/DHCP       | SLAAC + DHCPv6           |

### 1.3 Sub-redes: CIDR e VLSM

#### CIDR (Classless Inter-Domain Routing)

Permite dividir redes de forma flexível usando a notação de prefixo (`/`).

Exemplo:

- Rede: `192.168.10.0/24` → 256 endereços (254 hosts úteis).
- Máscara: `255.255.255.0`.

**Cálculo rápido de hosts:**

- Número de hosts úteis = \( 2^{(32 - prefixo)} - 2 \)

#### VLSM (Variable Length Subnet Mask)

Permite criar sub-redes de tamanhos diferentes a partir de uma mesma rede, otimizando o uso de endereços.

**Exemplo prático de VLSM:**

Rede original: `192.168.1.0/24`

Necessidades:

- Rede A: 60 hosts → `/26` (64 endereços)
- Rede B: 28 hosts → `/27` (32 endereços)
- Rede C: 12 hosts → `/28` (16 endereços)
- Rede D: 2 hosts (ponto a ponto) → `/30` (4 endereços)

Alocação sugerida:

``` cisco
192.168.1.0/26    → Rede A
192.168.1.64/27   → Rede B
192.168.1.96/28   → Rede C
192.168.1.112/30  → Rede D
```

**Dica de estudo:** Sempre aloque as maiores sub-redes primeiro.

---

## 2. Ethernet, Switching, VLANs, Trunking, STP e LACP

### 2.1 Ethernet e Switching

- **Ethernet** é o padrão dominante de camada 2 (IEEE 802.3).
- Switches operam na camada 2 e usam o endereço **MAC** para encaminhar frames.
- Tabela MAC (CAM table): aprende endereços de origem e encaminha para a porta correta.
- Tipos de encaminhamento:
  - **Store-and-forward** (mais comum e seguro)
  - Cut-through
  - Fragment-free

### 2.2 VLANs (Virtual LANs)

VLANs segmentam logicamente uma rede física, criando broadcast domains separados.

**Benefícios:**

- Segurança
- Redução de broadcast
- Organização por departamento/função
- Flexibilidade (mesmo switch físico, redes lógicas diferentes)

**Tipos de VLAN:**

- Data VLAN
- Voice VLAN
- Management VLAN
- Native VLAN (importante em trunks)

**Configuração básica (Cisco):**

```cisco
vlan 10
 name VENDAS
vlan 20
 name TI
vlan 30
 name GUEST

interface GigabitEthernet0/1
 switchport mode access
 switchport access vlan 10
```

### 2.3 Trunking (802.1Q)

Permite que um único link físico transporte tráfego de múltiplas VLANs.

- Protocolo: **IEEE 802.1Q** (tagging de 4 bytes no frame).
- Native VLAN: tráfego sem tag (deve ser a mesma nos dois lados).
- **Atenção:** Native VLAN mismatch é causa comum de problemas.

```cisco
interface GigabitEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
 switchport trunk native vlan 99
```

### 2.4 STP (Spanning Tree Protocol)

Evita loops em topologias redundantes de camada 2.

- Versões: STP (802.1D), RSTP (802.1w), MSTP (802.1s).
- Conceitos principais:
  - Root Bridge (eleito pelo menor Bridge ID)
  - Root Port, Designated Port, Blocked Port
  - Estados: Blocking → Listening → Learning → Forwarding

**Boas práticas em datacenter:**

- Preferir **RSTP** ou **MSTP**.
- Em leaf-spine moderno, muitos ambientes usam **MLAG** ou designs que minimizam a necessidade de STP clássico.

### 2.5 LACP (Link Aggregation Control Protocol)

Permite agregar múltiplos links físicos em um único link lógico (EtherChannel / Port-Channel).

**Vantagens:**

- Maior largura de banda
- Redundância
- Balanceamento de carga

```cisco
interface range GigabitEthernet0/1-2
 channel-group 1 mode active
interface Port-channel1
 switchport mode trunk
```

- `mode active` = LACP
- `mode on` = estático (sem negociação)

---

## 3. Roteamento, DNS, DHCP e NAT

### 3.1 Roteamento Estático

Configuração manual de rotas.

```cisco
ip route 192.168.20.0 255.255.255.0 10.0.0.2
ip route 0.0.0.0 0.0.0.0 203.0.113.1   ! rota default
```

**Quando usar:** redes pequenas, links ponto a ponto, ou como backup de protocolos dinâmicos.

### 3.2 OSPF (Open Shortest Path First) — Noções

- Protocolo de estado de link (IGP).
- Usa o algoritmo de Dijkstra (SPF).
- Áreas: Area 0 (backbone) é obrigatória.
- Métrica: custo (baseado na largura de banda).
- Mensagens: Hello, DBD, LSR, LSU, LSAck.

**Configuração básica:**

```cisco
router ospf 1
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 network 10.0.0.0 0.0.0.3 area 0
```

### 3.3 BGP (Border Gateway Protocol) — Noções

- Protocolo de roteamento externo (EGP) da Internet.
- Baseado em path vector.
- Usa AS (Autonomous System).
- Atributos importantes: AS-Path, Local Preference, MED, Next-Hop.
- Em datacenter moderno: frequentemente usado com EVPN para overlay.

### 3.4 DNS (Domain Name System)

Traduz nomes em endereços IP.

- Hierárquico e distribuído.
- Tipos de registros: A, AAAA, CNAME, MX, NS, PTR, TXT.
- Processo de resolução: consulta recursiva ou iterativa.
- Porta: UDP/53 (consultas), TCP/53 (transferências de zona).

### 3.5 DHCP (Dynamic Host Configuration Protocol)

Atribui automaticamente:

- Endereço IP
- Máscara
- Gateway
- DNS
- Outras opções

**Processo (DORA):**

1. **Discover** (cliente → broadcast)
2. **Offer** (servidor → cliente)
3. **Request** (cliente → servidor)
4. **Acknowledge** (servidor → cliente)

### 3.6 NAT (Network Address Translation)

Traduz endereços privados em públicos (e vice-versa).

**Tipos:**

- **Static NAT:** 1:1
- **Dynamic NAT:** pool de endereços públicos
- **PAT (NAT Overload):** muitos-para-um (mais comum)

Em datacenters modernos o uso de NAT é reduzido (preferência por IPv6 e endereçamento público interno quando possível), mas ainda é fundamental em muitos cenários.

---

## 4. Topologias de Datacenter: Leaf-Spine e VXLAN

### 4.1 Topologia Leaf-Spine

Arquitetura moderna de datacenter que substitui o clássico core-aggregation-access.

**Características:**

- **Leaf switches:** conectam servidores (ToR – Top of Rack).
- **Spine switches:** interconectam todos os leafs.
- Todo leaf se conecta a todos os spines (full mesh).
- Latência previsível e baixa (no máximo 2 hops entre qualquer servidor).
- Escalabilidade horizontal excelente.

**Vantagens:**

- Bandwidth consistente
- Fácil expansão (adicionar leaf ou spine)
- Suporte nativo a ECMP (Equal-Cost Multi-Path)

### 4.2 VXLAN (Virtual Extensible LAN)

Tecnologia de overlay que permite criar redes L2 sobre uma infraestrutura L3.

**Por que surgiu:**

- Limitação de 4096 VLANs no 802.1Q.
- Necessidade de estender L2 entre datacenters ou racks distantes.
- Multitenancy em cloud.

**Como funciona:**

- Encapsula frames Ethernet originais dentro de pacotes UDP.
- Usa **VNI** (VXLAN Network Identifier) de 24 bits → até ~16 milhões de segmentos.
- Endpoint: **VTEP** (VXLAN Tunnel Endpoint).

**Control Plane moderno:**

- BGP EVPN é o padrão de fato para distribuição de informações de reachability (MAC/IP).

**Resumo prático:**

- Underlay: rede L3 (normalmente OSPF ou BGP) com leaf-spine.
- Overlay: VXLAN + EVPN.

---

## 7. Prática: Montar Rede no Packet Tracer ou EVE-NG

### Objetivo da topologia

Criar uma rede com:

- 3 VLANs
- Roteamento inter-VLAN (Router-on-a-Stick ou L3 Switch)
- Servidor DHCP
- ACLs básicas

### Sugestão de topologia

```text
[PC-Vendas] ---- [Switch] ---- [Router/L3-Switch] ---- [Internet/Simulado]
[PC-TI]    ----/        \
[PC-Guest] ---------------
```

**VLANs sugeridas:**

| VLAN | Nome | Rede | Gateway |
| ------ | -------- | ----------------- | --------------- |
| 10 | Vendas | 192.168.10.0/24 | 192.168.10.1 |
| 20 | TI | 192.168.20.0/24 | 192.168.20.1 |
| 30 | Guest | 192.168.30.0/24 | 192.168.30.1 |

### Passos recomendados

1. Criar as VLANs no switch.
2. Configurar portas de acesso.
3. Configurar trunk entre switch e roteador/L3 switch.
4. Configurar subinterfaces (Router-on-a-Stick) ou SVIs (L3 Switch).
5. Configurar DHCP (no roteador ou servidor dedicado).
6. Criar ACLs:
   - Bloquear Guest de acessar a rede TI.
   - Permitir apenas HTTP/HTTPS da VLAN Vendas para a Internet.
7. Testar conectividade e isolamento.

**Comandos úteis de verificação:**

```cisco
show vlan brief
show interfaces trunk
show ip interface brief
show ip route
show access-lists
```

---

## 8. Prática: Captura e Análise de Tráfego no Wireshark

### Objetivos de captura

1. **DNS**
   - Filtrar: `dns`
   - Observar query e response.
   - Verificar tipos de registro (A, AAAA).

2. **DHCP**
   - Filtrar: `bootp` ou `dhcp`
   - Identificar as 4 mensagens do processo DORA.
   - Verificar opções oferecidas (mask, gateway, DNS).

3. **TCP Handshake**
   - Filtrar: `tcp.flags.syn == 1`
   - Observar: SYN → SYN-ACK → ACK
   - Analisar números de sequência e acknowledgment.

### Dicas de análise

- Use colunas personalizadas (Source, Destination, Protocol, Info).
- Aplique filtros de display.
- Siga o stream TCP (`Follow → TCP Stream`).
- Observe tempos (latência) e retransmissões.

**Exercício sugerido:**

1. Capture o processo completo de abertura de um site (DNS + TCP handshake + HTTP).
2. Identifique cada etapa e anote os endereços IP e portas envolvidos.

---

## 9. Revisão: 20 Exercícios de Sub-redes (sem consulta)

Resolva os exercícios abaixo **sem consultar** tabelas ou calculadoras. Depois confira.

### Exercícios

1. Qual a máscara e o número de hosts úteis de uma rede `/26`?
2. Qual o range de endereços de `192.168.5.0/27`?
3. Quantas sub-redes `/28` cabem em uma `/24`?
4. Qual o broadcast de `10.10.10.64/28`?
5. Qual o primeiro e o último host útil de `172.16.0.0/30`?
6. Converta a máscara `255.255.255.192` para CIDR.
7. Uma rede precisa de 100 hosts. Qual o menor prefixo possível?
8. Qual a rede de `192.168.1.130/25`?
9. Quantos hosts úteis tem uma `/29`?
10. Faça VLSM: a partir de `192.168.0.0/24`, crie sub-redes para 50, 20, 10 e 2 hosts.
11. Qual o endereço de rede de `10.1.1.100` com máscara `/22`?
12. Quantas sub-redes `/26` cabem em uma `/20`?
13. Qual a máscara para obter exatamente 14 hosts úteis?
14. Determine se `192.168.10.50` e `192.168.10.100` estão na mesma sub-rede `/25`.
15. Qual o próximo endereço de rede após `172.16.4.0/23`?
16. Uma empresa tem a rede `200.100.50.0/24`. Precisa de 4 sub-redes iguais. Qual o novo prefixo e as redes?
17. Qual o último host útil de `10.0.0.0/12`?
18. Converta `/21` para notação decimal pontuada.
19. Quantos bits de host restam em uma `/27`?
20. Projete: rede `10.20.0.0/16`. Crie 3 sub-redes: uma com 500 hosts, outra com 200 e outra com 50. Mostre as alocações.

**Respostas (para correção posterior):**

1. `/26` → máscara `255.255.255.192` → 62 hosts  
2. `192.168.5.0` até `192.168.5.31` (broadcast `.31`)  
3. 16 sub-redes  
4. `10.10.10.79`  
5. `172.16.0.1` e `172.16.0.2`  
6. `/26`  
7. `/25` (126 hosts)  
8. `192.168.1.128`  
9. 6 hosts  
10. Exemplo: `/26` (50), `/27` (20), `/28` (10), `/30` (2)  
11. `10.1.0.0`  
12. 64 sub-redes  
13. `/28`  
14. Não (`.50` está em `.0/25`, `.100` em `.128/25`)  
15. `172.16.6.0/23`  
16. `/26` → `200.100.50.0`, `.64`, `.128`, `.192`  
17. `10.15.255.254`  
18. `255.255.248.0`  
19. 5 bits  
20. Exemplo de alocação (maior primeiro):  
    - 500 hosts → `/23` (`10.20.0.0/23`)  
    - 200 hosts → `/24` (`10.20.2.0/24`)  
    - 50 hosts → `/26` (`10.20.3.0/26`)

---

## 10. Entrega: Topologia Documentada no GitHub

### O que deve conter o repositório

1. **README.md** com:
   - Descrição do projeto
   - Objetivo da topologia
   - Diagrama (pode ser imagem ou draw.io / mermaid)
   - Tabela de endereçamento
   - Lista de equipamentos e suas funções

2. **Pasta `/configs`** com os arquivos de configuração de cada dispositivo (switch, router, etc.).

3. **Diagrama da topologia** (formato PNG/SVG ou arquivo draw.io).

4. **Documentação de testes** (resultados de ping, traceroute, verificação de ACLs, etc.).

5. (Opcional) Scripts de automação ou arquivos de exportação do Packet Tracer / EVE-NG.

### Exemplo de estrutura de repositório

```text
rede-laboratorio/
├── README.md
├── diagrama/
│   └── topologia.png
├── configs/
│   ├── switch-core.cfg
│   ├── router-borda.cfg
│   └── ...
├── docs/
│   ├── enderecamento.md
│   └── testes.md
└── capturas/
    └── wireshark-exemplo.pcapng
```

### Dica de qualidade

- Use commits claros e descritivos.
- Inclua no README as tecnologias utilizadas (VLANs, OSPF, DHCP, ACLs, etc.).
- Documente qualquer particularidade ou problema encontrado e como foi resolvido.

---

## Roteiro de Estudo Sugerido

| Semana | Foco                                      | Atividade prática                     |
|--------|-------------------------------------------|---------------------------------------|
| 1      | OSI/TCP-IP + IPv4/IPv6 + Sub-redes        | 20 exercícios de sub-redes            |
| 2      | Switching, VLANs, Trunk, STP, LACP        | Configurar VLANs e trunks             |
| 3      | Roteamento + DHCP + DNS + NAT             | Router-on-a-Stick + DHCP              |
| 4      | Leaf-Spine + VXLAN (conceitos)            | Leitura + diagramas                   |
| 5      | Laboratório completo + Wireshark          | Montar topologia + capturas           |
| 6      | Documentação e entrega no GitHub          | Finalizar repositório                 |

---

**Boa sorte nos estudos!**  
Dominar esses fundamentos é o que diferencia um profissional de redes de alguém que apenas “sabe configurar comandos”.

Qualquer dúvida sobre algum tópico específico, peça aprofundamento.
