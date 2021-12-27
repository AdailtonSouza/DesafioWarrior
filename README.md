# DesafioWarrior
Desafio Warrior - Sistema de Vendas de Ingressos

## Escopo
Uma plataforma web que permite comprar ingressos. 

## Desenho Macro da Solução
### Base de dados
Azure CosmosDB - banco de dados onde serão armazenados as informações das sessões e os ingressos, 
nome da base de dados: Ingressos
Essa base de dados terá duas tabelas: Sessao e Ingressos
Estrutura da base de dados:
#### Tabela Sessao
- sessaoId: id da sessão - guid
- nome: nome da sessão - string
- filme: nome do filme - string
- local: local da sessão - string
- data: data da sessão (sem hora) - string formato data
- horario: horario da sessão - string formato hora
- status: status da sessão - string - valores válidos: EnPlanejamento, EnVendas, Encerrada)
- valorIngresso: valor do ingresso: - número decimal
- Lugares: lugares da sessão - objeto json com informação a identificação do lugar e o id do ingresso, para lugar já vendido ou null para lugar vago, exemplo: {
	A1: { ingressoid: guid }
	A1: null
A2: null
…
ZN: null
}

#### Tabela Ingresso
- ingressoId: id do ingresso - guid
- sessaoId: id da sessão - guid
- sessaoNome: nome da sessão - string
- filme: nome do filme - string
- local: local da sessão-  string
- data: data da sessão (sem hora) - string formato data
- horario: horario da sessão - string formato hora
- lugar: identificação do lugar - string
- usuarioID: identificação do usuário - guid
- usuarioName:nome do usuário - string
- valorIngresso: valor do ingresso: - número decimal
- paymentType: tipo de pagamento - string
- dataHora: data e hora da criação do ingresso - datetime

## Recursos de computação
- Container Azure Function: function com trigger HTTP responsável por gerenciar os ingressos.
- Contrainer: Nginx: serviço de gateway 
- Container: Keycloack - serviço de identificação
- Azure SignalR service - serviço de comunicação em tempo real
- Azure DevOps: configuração de pipelines CI/CD
- Cluster Kubernet 

O Azure function, Nginx e Keyclock serão instalados pela pipeline de CI/CD e serão orquestrados pelo Cluster Kubernet.
Obs.: as informações de usuário (id e nome) serão fornecidas pelo Keycloack

## Desenho micro da solução 
### Recursos técnicos 
#### Frameworks 
	ReactJs para o cliente

#### Linguagens 
	NodeJs para o server 

#### Ferramentas
	VsCode: para desenvolvimento
AzureDevOps e Azure Portal para adminstração e criação dos recursos na Azure

#### Detalhes da solução
O serviço de identificação ira determinar qual é o papel do usuário após o login: usuário comum ou adminstrador

A function Ingressos terá os seguintes métodos
- criarSessao - Cria a sessão
- alterarSessao - Altera as informações da sessão
- apagarSessao - Apaga a sessão
- listarSessao - lista as sessões válidas
- pegarSessao - recuperar uma sessão baseado no id
- criarIngressos - criar os ingressos baseados nos lugares selecionados de uma sessão

A aplicação cliente deve ter um modulo de cadastro da sessão feito pelo adminstrador do sistema e um módulo de vendas que será acessada pelo usuários.

O modulo de cadastro de sessão deve seguir as seguintes regras
Sessões com o status: EnVendas ou Encerrada não pode ser apagada.
Sessão com status EnVendas só pode ser Encerrada
Se o status de uma sessão for EnVendas ou Encerrada ele não pode ser alterados para EnPlanejamento.

O usuário comum irá acessar o modulo de vendas de ingresso. Esse modulo deve mostrar apenas sessões com o status EnVendas, e com data / horários válidos (não mostrar sessões com datas/horários já vencido). O usuário pode filtar as sessões por procurando por nome, por data ou ambos.

O método CriarIngresso dese seguir algumas regras:
Deve verificar se o numero de ingressos para criação é maior que 10, se sim deve retornar um erro.
Antes de gerar criar os ingressos deve verificar se os lugares selecionados já foram salvas, se sim deve retornar um erro
Gerar um ingresso para cada lugar selecionado pelo usuário utilizando as informações da sessão
Alterar o objeto sessão incluindo o id do ingresso nos lugares selecionados
Gravar as informações no banco de dados
Os ultimos três passos deve ser feitos atráves de uma transação, se ocorrer algum erro a transação deve ser desfeita e deve retornar um erro para o usuário.
Após a gravação dos dados o metodo deve:
Enviar uma mensagem para o recurso SignalR com o objeto da sala atualizado. A aplicação cliente vai utilizar esse objeto para atualizar a informação da sessão mostrando quais lugares já foram vendidos
Enviar os ingressos criados para o usuário

Obsevações:
O metodo CriarIngresso deve ser chamado apenas depois de confirmado o pagamento

A aplicação clente deve estar preparada para receber mensagem do serviço SignalR para atualizar as informações da sessão. Após a atualização da informação da sessão, deve verificar se algum lugar que o usuário selecionou foi vendido, se sim deve emitir uma mensagem se erro

A aplicação cliente também deve permitir que o usuário selecione apenas 10 ingressos antes de chamar o metodo de criar ingresso.



