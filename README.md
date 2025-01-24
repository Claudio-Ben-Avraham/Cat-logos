Gerenciador de Catálogos da Netflix com Azure Functions

Este projeto implementa um Gerenciador de Catálogos da Netflix usando Azure Functions para criar, ler, atualizar e excluir (CRUD) informações sobre filmes e séries. O banco de dados utilizado é o Azure Cosmos DB para garantir alta disponibilidade e escalabilidade.
Visão Geral

O Gerenciador de Catálogos permite gerenciar informações sobre filmes e séries, com funcionalidades como:

    Adicionar novos filmes/seriados ao catálogo.
    Atualizar informações de filmes e séries.
    Buscar informações sobre filmes e séries.
    Excluir filmes e séries do catálogo.

Tecnologias Utilizadas

    Azure Functions: Para criar APIs sem servidor para gerenciar o catálogo de filmes e séries.
    Azure Cosmos DB: Para armazenamento de dados de forma escalável e rápida.
    Azure Key Vault: Para gerenciamento seguro de segredos e configurações de conexão.
    Azure Logic Apps (opcional): Para orquestrar workflows, se necessário, como notificações ou processos complexos.

Estrutura do Projeto

O projeto é dividido em:

    Azure Functions: Implementação das APIs RESTful que interagem com o banco de dados.
    Banco de Dados: Um banco de dados Cosmos DB para armazenar as informações dos filmes e séries.
    Autenticação: Utilização do Azure Active Directory (opcional) para autenticação de usuários e acesso seguro.

Passos para Implantação
Passo 1: Criar Recursos no Azure

    Criar uma Conta no Azure: Se você ainda não tem uma conta, crie uma conta no Azure Portal.

    Criar o Banco de Dados Cosmos DB:
        No portal do Azure, acesse Criar um Recurso.
        Escolha Cosmos DB e siga as instruções para criar uma conta no Cosmos DB (preferencialmente utilizando a API SQL).

    Criar uma Function App:
        No portal do Azure, vá até Criar um Recurso e procure por Function App.
        Escolha um nome para a função, configure o plano de hospedagem e a região.

    Configurar o Azure Key Vault (opcional):
        Para armazenar segredos e configurações sensíveis (como strings de conexão), crie o Azure Key Vault.

    Configurar o Azure Active Directory (opcional):
        Para controlar o acesso às funções, configure a autenticação via Azure Active Directory.

Passo 2: Clonar o Repositório

Clone este repositório para o seu ambiente local:

git clone https://github.com/seuusuario/gerenciador-catalogo-netflix-azure-functions.git
cd gerenciador-catalogo-netflix-azure-functions

Passo 3: Configuração Local

    Instalar Ferramentas:
        Instale o Azure Functions Core Tools para testar as funções localmente.
        Instale o Azure CLI para gerenciar os recursos do Azure.
        Instale a extensão do Visual Studio Code para Azure Functions.

    Configuração de Variáveis de Ambiente:
        Crie um arquivo .env ou configure variáveis de ambiente para armazenar a string de conexão do Cosmos DB e outras configurações sensíveis.

COSMOS_DB_CONNECTION_STRING=seu_connection_string_do_cosmos_db
FUNCTIONS_WORKER_RUNTIME=dotnet

Passo 4: Implementar Funções

    Criar Função de Adicionar Filme/Serie:

[FunctionName("AdicionarFilme")]
public static async Task<IActionResult> AdicionarFilme(
    [HttpTrigger(AuthorizationLevel.Function, "post", Route = "filmes")] HttpRequestData req,
    FunctionContext executionContext)
{
    var filme = await req.ReadFromJsonAsync<Filme>();

    var cosmosClient = new CosmosClient(Configuration["COSMOS_DB_CONNECTION_STRING"]);
    var container = cosmosClient.GetContainer("CatalogoNetflix", "Filmes");

    await container.CreateItemAsync(filme);

    return new OkObjectResult(filme);
}

    Criar Função de Buscar Filmes/Series:

[FunctionName("BuscarFilmes")]
public static async Task<IActionResult> BuscarFilmes(
    [HttpTrigger(AuthorizationLevel.Function, "get", Route = "filmes/{id}")] HttpRequestData req,
    string id,
    FunctionContext executionContext)
{
    var cosmosClient = new CosmosClient(Configuration["COSMOS_DB_CONNECTION_STRING"]);
    var container = cosmosClient.GetContainer("CatalogoNetflix", "Filmes");

    var response = await container.ReadItemAsync<Filme>(id, new PartitionKey(id));

    return new OkObjectResult(response.Resource);
}

    Criar Função de Atualizar Filme/Serie:

[FunctionName("AtualizarFilme")]
public static async Task<IActionResult> AtualizarFilme(
    [HttpTrigger(AuthorizationLevel.Function, "put", Route = "filmes/{id}")] HttpRequestData req,
    string id,
    FunctionContext executionContext)
{
    var filme = await req.ReadFromJsonAsync<Filme>();

    var cosmosClient = new CosmosClient(Configuration["COSMOS_DB_CONNECTION_STRING"]);
    var container = cosmosClient.GetContainer("CatalogoNetflix", "Filmes");

    await container.UpsertItemAsync(filme, new PartitionKey(id));

    return new OkObjectResult(filme);
}

    Criar Função de Excluir Filme/Serie:

[FunctionName("ExcluirFilme")]
public static async Task<IActionResult> ExcluirFilme(
    [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "filmes/{id}")] HttpRequestData req,
    string id,
    FunctionContext executionContext)
{
    var cosmosClient = new CosmosClient(Configuration["COSMOS_DB_CONNECTION_STRING"]);
    var container = cosmosClient.GetContainer("CatalogoNetflix", "Filmes");

    await container.DeleteItemAsync<Filme>(id, new PartitionKey(id));

    return new OkResult();
}

Passo 5: Testando Localmente

Para testar as funções localmente, execute o seguinte comando:

func start

As funções estarão disponíveis localmente em http://localhost:7071.
Passo 6: Implantação no Azure

    Implantar Funções no Azure:

az functionapp deployment source config-zip \
    --resource-group seu-grupo-de-recursos \
    --name seu-function-app \
    --src ./publicado.zip

    Configurar Banco de Dados:
        No portal do Azure, conecte sua função ao Cosmos DB configurando a string de conexão no Application Settings.

Passo 7: Testar na Nuvem

Após a implantação, você pode testar as funções diretamente no endpoint gerado pelo Azure Function App.
Conclusão

Você agora tem um Gerenciador de Catálogos da Netflix em funcionamento usando Azure Functions e Cosmos DB. A arquitetura sem servidor garante escalabilidade, enquanto o Cosmos DB oferece um banco de dados altamente disponível e de baixo custo. Você pode expandir este projeto implementando autenticação, integração com outros serviços Azure, ou melhorias no fluxo de dados.
