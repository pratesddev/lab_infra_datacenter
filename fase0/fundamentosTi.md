# 📚 Material de Estudo – Fase 0: Fundamentos de TI

## Objetivo da fase

Compreender, de forma sólida e prática, como um computador funciona internamente e como o sistema operacional gerencia os recursos de hardware. Esta base é essencial para qualquer caminho em infraestrutura, cloud, DevOps, segurança ou desenvolvimento.

### 1️⃣ Entender: CPU, memória, armazenamento, BIOS/UEFI, binário e hexadecimal

- CPU (processador): o "cérebro" do computador. Executa instruções. Conceitos importantes: núcleos, threads, clock, cache (L1, L2, L3) e registradores;
- Memória (RAM): memória volátil de acesso rápido. É onde os programas e dados ficam enquanto estão sendo usados. Diferença entre RAM e armazenamento;
- Armazenamento: HD (mecânico) e SSD (flash). Persistente (não perde dados ao desligar). Diferença de velocidade, latência  e durabilidade;
- BIOS/UEFI: firmware que inicializa o hardare e carrega o sistema operacional. UEFI é a evolução moderna da BIOS (suporta discos maiores, interface gráfica, Secure Boot, etc).
- Binário e Hexadecimal:
  - Binario (base 2) → linguagem real da máquina (0 e 1);
  - Hexadecimal (base 16) → forma mais legível de representar valores binários (muito usado em endereços de memória, cores, dumps etc.).

### 2️⃣ Cursos recomendados

- Fundação Bradesco - Escola Virtual (Curso de Fundamentos de TI) - Concluído
- Microsoft Learn (trilhas em portugês sobre conceitos de computação e sistemas operacionais)

### 3️⃣ Leitura recomendada

- Livro: *Organização Estruturada de Computadores* - Andrew S. Tanenbaum - Capítulos introdutórios

### 4️⃣ Prática obrigatória

- Instalação de máquinas virtuais, uma Windows e uma Linux

### 5️⃣ Revisão - Caminho de um dado entre CPU, memória e disco

Em um computador moderno, um dado raramente vai direto do disco para a CPU. Ele percorre uma hierarquia de memórias com velocidades e capacidades muito diferentes. O objetivo dessa hierarquia é minimizar o tempo que a CPU fica esperando.

#### Disco (armazenamento secundário)

O disco (HDD ou SSD) é a memória mais lenta e de maior capacidade. Os dados ficam armazenados de forma persistente, mesmo com o computador desligado. Quando um programa precisa de um arquivo ou de uma página de memória virtual que não está na RAM, o sistema operacional solicita a leitura do disco. A transferência ocorre através do barramento (SATA, NVMe etc.) e o dado chega à memória principal (RAM) em blocos (páginas, normalmente de 4kb).

#### Memória principal (RAM)

A RAM é muito mais rápida que o disco, mas volátil. Uma vez que o dado chega à RAM, ele fica disponível para a CPU. O sistema operacional e o gerenciador de memória virtual controlam quais páginas estão residentes na RAM. Se a CPU precisa de um dado que ainda não está na RAM, ocorre uma page *fault* e o sistema busca o bloco no disco.

#### Caches da CPU (L1, L2 e L3)

Antes de chegar aos registradores da CPU, o dado passa pela hierarquia de cache:

- Cache L3 (compartilhada entre os núcleos): maior e um pouco mais lenta;
- Cache L2 (por núcleo ou compartilhada): intermediária;
- Cache L1 (por núcleo, separada em instruções e dados): a mais rápida e menor.

#### Registradores da CPU

Os registradores são a memória mais rápida e mais próxima da unidade de execução. O dado finalmente chega a um registrador, onde a ULA (Unidade Lógica e Aritimética) ou outras unidades funcionais podem processá-lo. Após o processamento, o resultado pode ser escrito de volta nos registradores, depois no cache e, eventualmente, na RAM (e, se necessário, no disco).

#### Fluxo resumido

Disco → RAM → Cache L3 → Cache L2 → Cache L1 → Registradores → CPU

#### Fluxo resumido (escrita)

CPU → Registradores → Cache → RAM → (posteriormente) Disco

#### Por que essa hierarquia existe?

Cada nível mais próximo da CPU é mais rápido, mas também mais caro e menor. O sistema explora a *localidade temporal* (dados usados recentemente tendem a ser usados de novo) e a *localidade espacial* (dados próximos em memória tendem a ser acessados juntos). Assim, a maioria dos acessos encontra o dado nos caches, e o disco só é acessado quando realmente necessário.

Em resumo, o caminho de um dado é uma sequência de cópias entre níveis de memória de velocidades crescentes, controlada pelo sistema operacional, pelo controlador de memória e pelos mecanismos de cache da CPU. Essa organização é o que permite que processadores modernos operem em velocidades de GHz mesmo com discos relativamente lentos.
