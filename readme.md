# LGit - Provedor de Controle de Código Fonte Git para IDEs (via API SCC)

## Visão Geral

LGit é um plugin de controle de código fonte (SCC) que integra a funcionalidade do Git em Ambientes de Desenvolvimento Integrado (IDEs) que suportam a API SCC (Source Code Control) da Microsoft. Ele atua como uma ponte, permitindo que o IDE execute operações Git comuns (como commit, push, pull, branch, merge, diff, histórico) através de sua interface de usuário padrão para controle de versão. O projeto utiliza a biblioteca `libgit2` para todas as interações com repositórios Git.

Este projeto parece ser mais antigo, utilizando arquivos de projeto do Visual Studio como `.dsp` e `.dsw`, e implementando a versão 1.3 da API SCC.

## Funcionalidades Principais

*   **Integração com IDEs via API SCC:** Permite que IDEs compatíveis gerenciem repositórios Git.
*   **Operações Git Fundamentais:**
    *   Inicializar (`init`) e Clonar (`clone`) repositórios.
    *   Adicionar (`add`) arquivos ao stage.
    *   Realizar Commit (`commit`) de alterações.
    *   Visualizar Status (`status`) de arquivos.
    *   Visualizar Histórico (`log`) de commits.
    *   Comparar Diferenças (`diff`) entre versões de arquivos ou commits.
    *   Gerenciar Branches (`branch`): criar, listar, trocar.
    *   Mesclar Branches (`merge`).
    *   Aplicar Patches (`apply`).
    *   (Potencialmente) Operações remotas como `fetch`, `pull`, `push` (a extensão exata depende da implementação dos callbacks remotos).
*   **Interface de Usuário:**
    *   Utiliza caixas de diálogo nativas do Windows para interações como clone, commit, seleção de branch, etc.
    *   Fornece um explorador Git independente (`LGitStandaloneExplorer`).
*   **Logging:** Capacidade de registrar informações de depuração para diagnóstico.

## Arquitetura

O LGit opera como uma DLL carregada pelo IDE. O fluxo de comunicação é geralmente:

1.  **IDE:** O usuário interage com as opções de controle de versão do IDE.
2.  **API SCC:** O IDE traduz essas ações em chamadas para as funções da API SCC (ex: `SccOpenProject`, `SccCommit`, `SccQueryInfo`).
3.  **LGit (Plugin):**
    *   Implementa as funções da API SCC (definidas em `scc_1_3.h`).
    *   Gerencia o contexto do projeto Git (`LGitContext`), incluindo o ponteiro para o repositório `libgit2`.
    *   Traduz as chamadas da API SCC e os parâmetros do IDE em operações `libgit2`.
    *   Converte dados entre formatos (ex: ANSI para UTF-8 para caminhos de arquivo).
    *   Apresenta caixas de diálogo para entrada do usuário quando necessário.
4.  **libgit2:** A biblioteca C executa as operações Git no repositório.
5.  **LGit (Plugin):** Retorna os resultados (ou erros) da `libgit2` para o IDE através da API SCC.

## Componentes Chave do Código

*   **`LGit.cpp` / `LGit.h`:** Ponto de entrada principal da DLL. Implementa funções de inicialização/desinicialização da API SCC (`SccInitialize`, `SccUninitialize`). `LGit.h` define estruturas centrais como `LGitContext` e macros de exportação.
*   **`project.cpp`:** Lida com a abertura (`SccOpenProject`), fechamento (`SccCloseProject`) e obtenção de informações do projeto Git (`SccGetProjPath`). Contém lógica para localizar ou inicializar repositórios.
*   **`query.cpp`:** Implementa `SccQueryInfo` e `SccPopulateList` para fornecer ao IDE o status dos arquivos (controlado, modificado, não versionado, etc.) usando `git_status_foreach_ext`.
*   **`caps.cpp`:** Implementa `SccGetCaps` e `SccGetExtendedCapabilities` para informar ao IDE sobre as funcionalidades suportadas pelo provedor.
*   **`commit.cpp` (e `commitmk.cpp`, `commitvw.cpp`):** Gerencia a lógica de commit (check-in), incluindo a interface do usuário para mensagens de commit.
*   **`clone.cpp`:** Implementa a funcionalidade de clonagem de repositórios remotos, incluindo uma caixa de diálogo.
*   **`diff.cpp` / `diffwin.cpp`:** Lida com a visualização de diferenças entre arquivos e revisões.
*   **`branch.cpp` / `merge.cpp`:** Fornece funcionalidades para gerenciamento e mesclagem de branches.
*   **`stage.cpp`:** Gerencia o "stage" (índice) do Git, permitindo adicionar e remover arquivos do stage.
*   **`history.cpp`:** Responsável por buscar e exibir o histórico de commits.
*   **`apply.cpp`:** Implementa a funcionalidade de aplicar patches a partir de arquivos `.diff` ou `.patch`.
*   **`path.cpp`:** Utilitários para manipulação de caminhos de arquivo, incluindo normalização e conversão entre separadores de diretório.
*   **`logging.cpp`:** Fornece a função `LGitLog` para registrar mensagens.
*   **`winutil.cpp`:** Funções utilitárias específicas para a GUI do Windows.
*   **`resource.h` / `LGit.rc`:** Definições de recursos da interface do usuário (caixas de diálogo, strings, ícones).
*   **`scc_1_3.h`:** Arquivo de cabeçalho que define a interface da API SCC da Microsoft (versão 1.3).

## Dependências

*   **API SCC da Microsoft (versão 1.3):** Para integração com o IDE.
*   **libgit2:** Biblioteca C para interagir com repositórios Git. As funções `git_*` são usadas extensivamente.

## Compilando o Projeto (Suposição)

O projeto parece ser configurado para ser compilado com uma versão mais antiga do Microsoft Visual Studio (dado os arquivos `.dsp` e `.dsw`).

1.  **Pré-requisitos:**
    *   Microsoft Visual Studio (versão compatível com arquivos `.dsp`, ex: VC++ 6.0 ou Visual Studio .NET 2002/2003).
    *   Biblioteca `libgit2` (cabeçalhos e arquivos `.lib`) devidamente configurada nas opções do projeto (include paths, library paths).
2.  **Compilação:**
    *   Abrir o arquivo de workspace `LGit.dsw` no Visual Studio.
    *   Selecionar a configuração de compilação desejada (ex: Debug, Release).
    *   Compilar o projeto (Build Solution).

A saída principal será uma DLL (provavelmente `LGit.dll`) que pode ser registrada e usada por IDEs compatíveis com SCC.

## Licenciamento

Este projeto é licenciado sob a **GNU General Public License (GPL), versão 2**, conforme indicado no arquivo `COPYING.txt`.

## Considerações para Desenvolvimento Futuro

*   **Modernização:** Atualizar o projeto para usar versões mais recentes do Visual Studio e, possivelmente, uma API de integração mais moderna se o objetivo for suportar IDEs mais novos.
*   **Tratamento de Strings:** Garantir o tratamento robusto de codificação de caracteres entre o IDE (geralmente ANSI/MBCS em IDEs mais antigos) e `libgit2` (UTF-8).
*   **Gerenciamento de Erros:** Melhorar o mapeamento de erros da `libgit2` para códigos de erro SCC e fornecer feedback claro ao usuário.
*   **Operações Remotas:** Expandir e robustecer o suporte para operações remotas (fetch, pull, push), incluindo autenticação e tratamento de progresso.
*   **Testes:** Implementar um conjunto de testes unitários e de integração para garantir a estabilidade.