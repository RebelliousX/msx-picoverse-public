# Projeto MSX PicoVerse 2040

|PicoVerse Frontal|PicoVerse Traseira|
|---|---|
|![PicoVerse Frontal](/images/2025-12-02_20-05.png)|![PicoVerse Traseira](/images/2025-12-02_20-06.png)|

O PicoVerse 2040 é um cartucho para MSX baseado no Raspberry Pi Pico que usa firmware substituível para estender as capacidades do computador. Ao carregar diferentes imagens de firmware, o cartucho pode executar jogos e aplicações MSX e emular dispositivos de hardware adicionais (como mappers de ROM, RAM extra ou interfaces de armazenamento), adicionando periféricos virtuais ao MSX. Um desses firmwares é o sistema MultiROM, que fornece um menu na tela para navegar e iniciar múltiplos títulos ROM armazenados no cartucho.

O cartucho também pode expor a porta USB‑C do Pico como dispositivo de armazenamento em massa, permitindo copiar ROMs, DSKs e outros arquivos diretamente de um computador para o cartucho.

Além disso, é possível usar uma variante de firmware Nextor com suporte a mapper de memória +240 KB para aproveitar totalmente o hardware PicoVerse 2040 em sistemas MSX com memória limitada.

Recursos disponíveis na versão atual do cartucho PicoVerse 2040:

* Sistema de menu MultiROM para selecionar e iniciar ROMs MSX.
* Suporte ao Nextor OS com mapper de memória opcional de +240 KB.
* Suporte a dispositivo de armazenamento em massa USB para carregar ROMs e DSKs.
* Suporte para vários mappers MSX (PL-16, PL-32, KonSCC, Linear, ASC-08, ASC-16, Konami, NEO-8, NEO-16).
* Compatibilidade com sistemas MSX, MSX2 e MSX2+.

## Manual do Criador de UF2 MultiROM

Esta seção documenta a ferramenta de console `multirom` usada para gerar imagens UF2 (`multirom.uf2`) que gravam o cartucho PicoVerse 2040.

Por padrão, a ferramenta `multirom` varre arquivos ROM MSX em um diretório e os empacota em uma única imagem UF2 que pode ser gravada no Raspberry Pi Pico. A imagem resultante normalmente contém o binário do firmware do Pico, a ROM do menu MultiROM, uma área de configuração que descreve cada entrada ROM e as cargas das ROMs.

Dependendo das opções, `multirom` também pode produzir UF2 que inicializam diretamente em um firmware personalizado em vez do menu MultiROM. Por exemplo, é possível gerar UF2 com firmware que implemente o Nextor. Nesse modo, a porta USB‑C do Pico pode ser usada como armazenamento em massa para carregar ROMs e DSKs a partir do Nextor (por exemplo, via SofaRun).

## Visão geral

Se executado sem opções, a ferramenta `multirom.exe` varre o diretório de trabalho atual à procura de arquivos ROM MSX (`.ROM` ou `.rom`), analisa cada ROM para tentar adivinhar o tipo de mapper, constrói uma tabela de configuração descrevendo cada ROM (nome, byte do mapper, tamanho, offset) e incorpora essa tabela em um fragmento da ROM de menu MSX. A ferramenta concatena o blob de firmware do Pico, o fragmento da ROM de menu, a área de configuração e as ROMs, e serializa toda a imagem em um arquivo UF2 chamado `multirom.uf2`.

O arquivo UF2 (normalmente `multirom.uf2`) pode então ser copiado para o dispositivo de armazenamento em massa USB do Pico para gravar a imagem combinada. É preciso conectar o Pico mantendo pressionado o botão BOOTSEL para entrar no modo UF2. Uma nova unidade USB chamada `RPI-RP2` aparecerá e você pode copiar `multirom.uf2` para ela. Após a cópia, desconecte o Pico e insira o cartucho no seu MSX para inicializar o menu MultiROM.

Os nomes dos arquivos ROM são usados para nomear as entradas no menu MSX. Há um limite de 50 caracteres por nome. Um efeito de rolagem é usado para mostrar nomes mais longos no menu MSX, mas se o nome exceder 50 caracteres será truncado.

Se quiser usar o Nextor com seu PicoVerse 2040, deve executar a ferramenta com a opção `-s1` ou `-s2` para incluir a ROM Nextor embutida na imagem. A opção `-s1` inclui a ROM Nextor padrão sem suporte a mapper, enquanto `-s2` inclui uma ROM Nextor com suporte a mapper +240 KB. A ROM Nextor embutida será o único firmware carregado no boot, e o menu MultiROM não estará disponível. Você poderá então usar o SofaRun para carregar ROMs e DSKs a partir do armazenamento em massa USB do Pico.

Nota importante: Para usar um pen drive USB padrão pode ser necessário um adaptador OTG ou cabo para converter a porta USB‑C para USB‑A fêmea.

## Uso por linha de comando

Apenas executáveis do Microsoft Windows são fornecidos por enquanto (`multirom.exe`).

### Uso básico:

```
multirom.exe [options]
```

### Opções:
- `-n`, `--nextor`: Inclui a ROM NEXTOR embutida da configuração na saída. Esta opção ainda é experimental e por enquanto funciona apenas em modelos MSX2 específicos.
- `-h`, `--help`: Mostra a ajuda e sai.
- `-o <filename>`, `--output <filename>`: Define o nome do arquivo UF2 de saída (padrão `multirom.uf2`).
- `-s1`: O cartucho inicializa com Sunrise IDE Nextor (sem mapper de memória).
- `-s2`: O cartucho inicializa com Sunrise IDE Nextor (+240 KB de mapper de memória).
- Se nem `-s1` nem `-s2` forem especificados, a ferramenta produz uma imagem MultiROM com o menu.
- Se precisar forçar um tipo de mapper específico para um arquivo ROM, pode anexar uma tag de mapper antes da extensão `.ROM` no nome do arquivo. A tag não diferencia maiúsculas de minúsculas. Por exemplo, nomear um arquivo `Knight Mare.PL-32.ROM` força o uso do mapper PL-32 para essa ROM. Tags como `SYSTEM` são ignoradas.

### Exemplos
- Produz `multirom.uf2` com o menu MultiROM e todas as `.ROM` no diretório atual:
```
multirom.exe
```

- Produz `multirom.uf2` com a ROM Sunrise IDE Nextor (sem mapper de memória):
```
multirom.exe -s1
```
- Produz `multirom.uf2` com a ROM Sunrise IDE Nextor (+240 KB de mapper de memória):
```
multirom.exe -s2
```

## Como funciona (alto nível)

1. A ferramenta varre o diretório de trabalho atual procurando arquivos com extensão `.ROM` ou `.rom`. Para cada arquivo:
   - Extrai um nome de exibição (nome do arquivo sem extensão, truncado a 50 caracteres).
   - Obtém o tamanho do arquivo e valida que esteja entre `MIN_ROM_SIZE` e `MAX_ROM_SIZE`.
   - Chama `detect_rom_type()` para determinar heurísticamente o byte de mapper a usar na entrada de configuração. Se houver uma tag de mapper no nome do arquivo, esta substitui a detecção.
   - Se a detecção falhar, o arquivo é ignorado.
   - Serializa o registro de configuração por ROM (nome de 50 bytes, 1 byte de mapper, 4 bytes de tamanho LE, 4 bytes de offset em flash LE) na área de configuração.
2. Após a varredura, a ferramenta concatena (na ordem): o binário de firmware do Pico embutido, uma fatia inicial da ROM do menu MSX (`MENU_COPY_SIZE` bytes), a área completa de configuração (`CONFIG_AREA_SIZE` bytes), a ROM NEXTOR opcional e então as cargas das ROMs descobertas na ordem de descoberta.
3. A carga combinada é escrita como um arquivo UF2 chamado `multirom.uf2` usando `create_uf2_file()` que produz blocos UF2 de 256 bytes direcionados ao endereço de flash do Pico `0x10000000`.

## Heurísticas de detecção de mappers
- `detect_rom_type()` implementa uma combinação de verificações de assinatura (cabeçalho "AB", tags `ROM_NEO8` / `ROM_NE16`) e varredura heurística de opcodes e endereços para escolher mappers MSX comuns, incluindo (mas não limitado a):
  - Plain 16KB (mapper byte 1) — verificação de cabeçalho 16KB AB
  - Plain 32KB (mapper byte 2) — verificação de cabeçalho 32KB AB
  - Linear0 mapper (mapper byte 4) — verificação especial de layout AB
  - NEO8 (mapper byte 8) e NEO16 (mapper byte 9)
  - Konami, Konami SCC, ASCII8, ASCII16 e outros via pontuação ponderada
- Se nenhum mapper puder ser detectado com confiabilidade, a ferramenta ignora a ROM e reporta "unsupported mapper". Você pode forçar um mapper através da tag no nome do arquivo.
- Apenas os seguintes mappers são suportados na área de configuração e menu: `PL-16,  PL-32,  KonSCC,  Linear,  ASC-08,  ASC-16,  Konami,  NEO-8,  NEO-16`

## Uso do seletor de ROM MSX (menu)

Ao ligar o MSX com o cartucho PicoVerse 2040 inserido, o menu MultiROM aparece, mostrando a lista de ROMs disponíveis. Você pode navegar no menu usando as teclas de seta do teclado.

Use as setas para Cima e Para Baixo para mover o cursor de seleção pela lista de ROMs. Se você tiver mais de 19 ROMs, use as setas Esquerda e Direita para rolar páginas de entradas.

Pressione Enter ou Espaço para iniciar a ROM selecionada. O MSX tentará inicializar a ROM usando as configurações de mapper apropriadas.

A qualquer momento dentro do menu, você pode pressionar a tecla H para ler a tela de ajuda com instruções básicas. Pressione qualquer tecla para retornar ao menu principal.

## Usando Nextor com o cartucho PicoVerse 2040

Se você gravou o cartucho com firmware Nextor (usando `-s1` ou `-s2`), o cartucho inicializará diretamente no Nextor em vez do menu MultiROM. Essas opções comunicam-se com o MSX usando E/S mapeada em memória, exigindo que o cartucho seja inserido em um slot de cartucho padrão.

Uma vez que o Nextor esteja em execução, você pode usar o SofaRun ou qualquer outro inicializador compatível com Nextor para carregar ROMs e DSKs do armazenamento em massa USB do Pico. Pode ser necessário um adaptador ou cabo OTG para conectar pendrives USB-A padrão à porta USB‑C do cartucho.

Em sistemas MSX2 ou MSX2+ com a versão Nextor +240 KB (opção S2), o cartucho adicionará a memória extra ao sistema e a sequência de boot do BIOS refletirá o aumento de RAM. Alguns modelos MSX exibem a memória expandida total durante o boot.

A opção `-n` da ferramenta multirom inclui uma ROM Nextor experimental embutida que pode ser usada em modelos MSX2 específicos. No entanto, esta opção ainda está em desenvolvimento e pode não funcionar em todos os sistemas. Esta versão usa um método diferente (baseado em portas IO) para comunicar-se com o MSX, permitindo seu uso em sistemas onde o slot de cartucho não é padrão/expandido.

Mais detalhes sobre o protocolo usado para comunicar-se com o computador MSX podem ser encontrados em Nextor-Pico-Bridge-Protocol.md. Os detalhes da comunicação com o firmware Sunrise IDE Nextor estão em Sunrise-Nextor.md.

### Como preparar um pendrive para Nextor

1. Conecte o pendrive ao seu PC.
2. Crie uma partição FAT16 com no máximo 4GB no pendrive. Você pode usar ferramentas do sistema operacional ou software de particionamento de terceiros.
3. Copie os arquivos do sistema Nextor para o diretório raiz da partição FAT16. Você pode obter os arquivos do Nextor na distribuição oficial ou repositório. Você também precisa do arquivo COMMAND2 para o shell de comandos:
   1. [Nextor Download Page](https://github.com/Konamiman/Nextor/releases)
   2. [Command2 Download Page](http://www.tni.nl/products/command2.html)
4. Copie quaisquer ROMs MSX (`.ROM`) ou imagens de disco (`.DSK`) que deseja usar para o diretório raiz do pendrive.
5. Instale o SofaRun ou outro inicializador compatível com Nextor no pendrive se pretende usá-lo para iniciar ROMs e DSKs. Você pode baixar o SofaRun aqui: [SofaRun](https://www.louthrax.net/mgr/sofarun.html)
6. Ejete o pendrive com segurança.

## Ideias de aperfeiçoamento
- Melhorar as heurísticas de detecção de ROM para cobrir mais mappers e casos extremos.
- Implementar uma tela de configuração para cada entrada ROM (nome, sobrescrição de mapper, etc).
- Adicionar suporte a mais mappers de ROM.
- Implementar um menu gráfico com melhor navegação e informações sobre as ROMs.
- Adicionar suporte para salvar/carregar a configuração do menu para preservar preferências.
- Suportar arquivos DSK também, com entradas de configuração adequadas.
- Melhorar a detecção de mappers para cobrir mais casos.
- Adicionar suporte a slices de ROM de menu ou temas personalizados.
- Permitir o download de ROMs a partir de URLs e embuti-las diretamente.
- Suporte sem fio (wifi) via módulo ESP-01 externo.
- Permitir o uso do joystick para navegar no menu.
- Suporte para compactar ROMs para economizar espaço.

## Problemas conhecidos

- A inclusão da ROM Nextor embutida ainda é experimental e pode não funcionar em todos os modelos MSX2.
- Algumas ROMs com mappers incomuns podem não ser detectadas corretamente e serão ignoradas a menos que um mapper válido seja forçado via tag.
- Atualmente a ferramenta suporta apenas executáveis do Windows. Versões para Linux e macOS não estão disponíveis.
- A ferramenta não valida a integridade das ROMs além de tamanho e verificações básicas de cabeçalho. ROMs corrompidas podem causar comportamento inesperado.
- O menu MultiROM não suporta arquivos DSK; apenas ROMs são listadas e iniciadas.
- A ferramenta não suporta subdiretórios; apenas processa ROMs no diretório de trabalho atual.
- A memória flash do Pico pode desgastar-se após muitos ciclos de escrita. Evite regravações excessivas.

## Modelos MSX testados

| Modelo | Tipo | Estado | Comentários |
| --- | --- | --- | --- |
| Adermir Carchano Expert 4 | MSX2+ | OK | Verificado |
| Gradiente Expert | MSX1 | OK | Verificado |
| JFF MSX | MSX1 | OK | Verificado |
| MSX Book | MSX2+ (clone FPGA) | OK | Verificado |
| MSX One | MSX1 | Not OK | Cartucho não reconhecido |
| National FS-4500 | MSX1 | OK | Verificado |
| Panasonic FS-A1GT | TurboR | OK | Verificado |
| Panasonic FS-A1ST | TurboR | OK | Verificado |
| Panasonic FS-A1WX | MSX2+ | OK | Verificado |
| Panasonic FS-A1WSX | MSX2+ | OK | Verificado |
| Sharp HotBit HB8000 | MSX1 | OK | Verificado |
| SMX-HB | MSX2+ (clone FPGA) | OK | Verificado |
| Sony HB-F1XD | MSX2 | OK | Verificado |
| Sony HB-F1XDJ | MSX2 | OK | Verificado |
| Sony HB-F1XV | MSX2+ | OK | Verificado |
| TRHMSX | MSX2+ (clone FPGA) | OK | Verificado |
| uMSX | MSX2+ (clone FPGA) | OK | Verificado |
| Yamaha YIS604 | MSX1 | OK | Verificado |

A versão Sunrise Nextor +240K foi testada em:

| Modelo | Estado | Comentários |
| --- | --- | --- |
| MSX1 | | |
| Sharp HotBit HB8000 (MSX1) | Not OK | O mapper de memória não funciona e o Nextor não é funcional |
| MSX2+ | | |
| TRHMSX (MSX2+, clone FPGA) | OK | Verificado |
| uMSX (MSX2+, clone FPGA) | OK | Verificado |

Autor: Cristiano Almeida Goncalves
Última atualização: 12/20/2025
