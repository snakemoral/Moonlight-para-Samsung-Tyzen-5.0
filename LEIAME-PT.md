# Moonlight NaCl Plus (Samsung Tizen 5.0)

Versão do fork `OneLiberty/moonlight-tizen-nacl` (a única que roda em TVs Samsung com Tizen 5.0,
como a Q60R) com recursos trazidos do app atual `OneLiberty/moonlight-chrome-tizen`.

O vídeo e o áudio continuam usando o plugin NaCl, igual ao app que você já instalou.
**O desempenho do stream não muda.** O que mudou é a interface e os recursos ao redor.

## O que foi portado

| Recurso | Como usar |
|---|---|
| **Wake-on-LAN** | Cartão do host → botão ☰ (ou botão **Start** do controle) → *Wake PC (WOL)* |
| **Cache das capas dos jogos** | Automático. A lista de jogos abre mais rápido da segunda vez em diante |
| **Atualizar capas** | Menu ☰ do host → *Refresh box art* |
| **Menu de configurações por host** | Botão ☰ no canto do cartão (Wake PC, Refresh box art, Remove) |
| **Confirmar antes de sair** | Botão **Voltar** (ou **B** no controle) na tela de hosts |
| **Adicionar host por campos de IP** | Setas ↑/↓ mudam o número, ←/→ trocam de campo. **Select** (controle) alterna para digitar IP ou nome |
| **Repetição ao segurar o direcional** | Segurar o direcional do controle repete o movimento |
| **Modelo da TV / versão na tela** | Canto inferior esquerdo |
| **Textos do pareamento com Sunshine** | PIN e instruções para o Sunshine e para o GeForce Experience |
| **Padrão 1080p, 60 FPS, 20 Mbps** | Só vale quando ainda não há configuração salva |
| **Destaque visual do item selecionado** | Botões e campos ganham contorno azul |
| **Teste de controle** | Ícone de gamepad no topo, ou botão **azul** do controle remoto da TV. Mostra o que a TV e o plugin enxergam do controle |

Correções feitas no caminho:

- O botão **Select** do controle dava erro (a função nunca foi ligada).
- Falhar o pareamento ao adicionar um host dava erro de JavaScript.
- O botão Continuar do "Add Host" podia ser acionado várias vezes seguidas.

## O que NÃO foi portado (e por quê)

Estes recursos do app atual dependem da API de vídeo WASM da Samsung, que só existe a partir do
Tizen 5.5. Na sua TV (5.0) não há como ligá-los:

- Escolha de codec (H.265, AV1) e HDR
- Frame pacing e Audio Sync
- Emulação de mouse pelo controle (depende do código nativo do app novo)

Também **não** atualizei o `moonlight-common-c`/Opus: não consigo compilar com o SDK da Samsung
aqui e uma troca dessas poderia quebrar o app sem aviso.

## Como compilar e instalar

Precisa de Docker no PC, e a TV com o **Modo Desenvolvedor** ligado apontando para o IP do PC.

```powershell
cd moonlight-tizen-nacl-plus
docker build -t moonlight-nacl-plus .
docker run -it --rm moonlight-nacl-plus
```

A compilação baixa o Tizen Studio e o SDK da Samsung, então demora e precisa de internet.
Dentro do container que abriu:

```
sdb connect IP_DA_TV
tizen install -n MoonlightNaCl.wgt
exit
```

O app usa o mesmo identificador do anterior, então **substitui** a versão instalada. Se o
instalador reclamar, apague o Moonlight da TV antes (segure o botão do app → Excluir) e instale de
novo. Talvez seja preciso parear de novo com o PC.

**Para instalar pelo Apps2Samsung** (*Custom WGT File*), tire o arquivo do container:

```powershell
docker create --name mlx moonlight-nacl-plus
docker cp mlx:/home/moonlight/MoonlightNaCl.wgt .
docker rm mlx
```

## Compilar pelo GitHub (sem Docker no PC)

O projeto inclui o workflow `.github/workflows/build-wgt.yml`, que compila e entrega o `.wgt` pronto.

1. Crie uma conta no GitHub e um repositório **vazio** (pode ser privado), sem README.
2. No PC, dentro da pasta do projeto (precisa do [Git](https://git-scm.com/downloads) instalado):

   ```
   git init -b main
   git add .
   git commit -m "Moonlight NaCl Plus"
   git remote add origin https://github.com/SEU_USUARIO/NOME_DO_REPOSITORIO.git
   git push -u origin main
   ```

   (O envio pelo navegador não serve: ele aceita no máximo 100 arquivos por vez e o projeto tem mais de 600.)
3. No repositório, abra a aba **Actions**. A compilação começa sozinha e leva de 10 a 30 minutos.
   Se não começar, escolha *Build MoonlightNaCl.wgt* e clique em **Run workflow**.
4. Quando ficar verde, abra a execução e baixe **MoonlightNaCl-wgt** em *Artifacts*. É um zip com o
   `MoonlightNaCl.wgt` dentro.
5. Instale pelo **Apps2Samsung** (*Custom WGT File*), ou com `sdb connect IP_DA_TV` e
   `tizen install -n MoonlightNaCl.wgt`.

Se a execução ficar vermelha, abra o passo que falhou e copie as últimas linhas do log para eu corrigir.

## Se algo der errado

- **Erro na compilação (`docker build`)**: copie as últimas ~30 linhas da saída e me mande. A única
  parte nova em C++ é o Wake-on-LAN, no final do `main.cpp`.
- **Erro ao instalar na TV**: abra o `config.xml` e apague as duas linhas `filesystem.read` e
  `filesystem.write`. O app funciona igual, só perde o cache das capas.
- **Wake-on-LAN não acorda o PC**: o PC precisa ter Wake-on-LAN ativado na BIOS e na placa de rede,
  e o app precisa já ter conectado a ele uma vez para saber o endereço MAC. Se o PC estiver só
  em Wi-Fi, o Wake-on-LAN normalmente não funciona.

## O que foi testado

- Interface: 31 testes automatizados num navegador simulando a TV (menus, diálogos, entrada de IP,
  cache de capas com e sem armazenamento, repetição do controle, Wake-on-LAN). Todos passaram.
- Wake-on-LAN, parte nativa: o pacote foi gerado e recebido corretamente em teste local.
- **Não testado na TV real**, e **não compilado com o toolchain da Samsung**. O cache de capas e o
  Wake-on-LAN dependem do firmware da TV e podem se comportar diferente.

As diferenças em relação ao fork original estão no histórico do Git.
