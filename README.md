# CalculadoraFreelancer05

Se trata da continuação do app <a href="https://github.com/dayaneLima/CalculadoraFreelancer04">CalculadoraFreelancer04</a>

## Arrumando nosso repositório

Vamos agora arrumar o nosso repositório. Utilizaremos Injeção de dependência juntamente com o padrão Repository.

### Interface Repository

Dentro do projeto CalculadoraFreelancer.Domain vamos criar uma pasta chamada Interface. Dentro desta pasta vamos criar uma interface chamada IRepository. 
Essa interface terá as assinaturas dos métodos básicos que é necessário para manipulação de dados. Primeiramente vamos criar a interface:

```c#
  public interface IRepository
  {
  }
````

Cada entidade terá seu repositório, então nossa interface poderá já definir que deverá ser de uma entidade específica, e utilizar dessa tipagem em seus métodos.
Vamos então definir uma tipagem genérica que depois em nossos repositórios específicos irá definir a tipagem concreta.
Então mude o nome da interface dessa forma:

```c#
  public interface IRepository<TEntity>
  {
  }
````

Agora vamos criar as funções básicas:

```c#
    public interface IRepository<TEntity>
    {
        Task<TEntity> Find(string id);
        Task<TEntity> GetFirst();
        Task<IEnumerable<TEntity>> GetAll();
        Task Insert(TEntity tEntity);
        Task Update(TEntity tEntity);
        Task Delete(TEntity tEntity);
    }
  ````
  
  ### Interface IProfissionalRepository
  
  Agora que temos nossa interface genérica, vamos criar uma interface para cada domínio que temos. ainda dentro do projeto CalculadoraFreelancer.Domain, na pasta profissionais 
  crie uma pasta chamada Repository, e dentro dela crie uma interface chamada IProfissionalRepository, dessa forma:
  
```c#
 public interface IProfissionalRepository
  {
  }
````
  
  Como temos uma interface base, chamada IRepository, isso significa que todas as outras interfaces de repositório devem implementar dela, pois ela possui as assinaturas das operações base. 
  Então vamos implementar de IRepository.
    
```c#
 public interface IProfissionalRepository : IRepository
  {
  }
````

Lembra que nossa interface IRepository tem uma tipagem (IRepository<TEntity>) ? É agora que vamos utilizar isso e ficará mais fácil de entender.
Como estamos criando agora o repositório do Profissional, essa tipagem do repositório será do Profissional, ou seja, agora todas aquelas assinaturas de funções retornarão e receberão objetos do tipo Profissinal.
A interface então ficará dessa forma:

```c#
    public interface IProfissionalRepository : IRepository<Profissional>
    {
    }
````
 
 Como ainda não temos nenhuma operação específica para o Profissional, nosso repositória ficará como mostrado acima.
 
   ### Interface IProjetoRepository
 
 Agora vamos fazer o mesmo para o Projeto. Ainda no projeto CalculadoraFreelancer.Domain, dentro da pasta Projetos crie uma pasta chamada Repository.
 Dentro dela crie uma interface chamada IProjetoRepository, implemente de IRepository definindo a tipagem para Projeto. Ficará assim:
 
 ```c#
public interface IProjetoRepository : IRepository<Projeto>
{
}
````

Perfeito, agora temos em nosso domínio todas as assinaturas base para manipulação de dados. 
O reuso e o ganho com isso é enorme, pois agora como há as operações base, quem desejar implementar o acesso a dados para o nosso domínio basta implementar dessas interfaces. 
Nossa camada de domínio não precisa saber como o acesso a dados é feito, se é MySql, Azure, SQLite, arquivo de texto, 
ele só se importa em chamar os métodos definidos na interface e receber o resultado especificado.

## Camada CalculadoraFreelancer.Infra.Data

A implementação do acesso a dados é feito na camada CalculadoraFreelancer.Infra.Data, vamos corrigí-la.

### Repositório Base

Você deve ter reparado que as classes AzureRepository e AzureProjetoRepository são praticamente iguais, 
mas uma possui mais métodos que outra, fizemos isso por questões didáticas, agora vamos arrumar essa bagunça.

Exclua a nossa classe AzureProjetoRepository, pois utilizaremos somente a AzureRepository.

A AzureRepository será a nossa classe base e genérica. Então primeiramente ela terá uma tipagem genérica relacionada a ela, como fizemos no IRepository. Ficará assim:

```c#
  public class AzureRepository<TEntity>
  {
      ...
  }
````

Como  ela será a nossa genérica, ela poderá implementar de IRepository, dessa forma:

```c#
 public class AzureRepository<TEntity> : IRepository<TEntity>
 {
    ...
 }
````

Todas as nossas Entidades herdam de Entity, podemos então definir em nossa classe que AzureRepository que o TEntity será do tipo Entity,  dessa forma:

```c#
    public class AzureRepository<TEntity> : IRepository<TEntity> where TEntity : Entity
    {
        ...
    }
````

Agora que corrigimos a assinatura da classe, vamos corrir suas propriedades e métodos.
Primeiramente a nossa propriedade Table não será mais do tipo Profissional e sim do nosso tipo genérico TEntity, ficando dessa forma:


```c#
     private IMobileServiceTable<TEntity> Table;
````
 
 No contrutor da classe AzureRepository onde instanciamos essa propriedade, devemos corrigir também a tipagem, no lugar de Profissional vamos trocar por TEntity:
 
 ```c#
   public AzureRepository()
  {
      string MyAppServiceURL = "sua url aqui";
      Client = new MobileServiceClient(MyAppServiceURL);
      Table = Client.GetTable<TEntity>();
  }
````
 
 Vamos as correções dos métodos agora. Primeiramente altere os métodos Insert e Delete, o tipo de retorno deles é void, mude para Task, pois será operações assíncronas.
 
 Agora tudo que é do tipo Profissional deverá ser alterado para ser do tipo TEntity, vamos acelerar essa correção. Aperte ctr + h, abrirá essa telinha:
 
 <img src="https://github.com/dayaneLima/CalculadoraFreelancer0t/blob/master/Docs/Imgs/aula_05_subistituir_01.png" alt="Subistituir texto" width="100%">
 
 No primeiro campo digite o termpo que quer encontrar, no caso, Profissional, no segundo campo digite por qual palavra deseja substituir, no caso coloque TEntity. 
 Após clique para substituir todos, dessa forma:
 
 <img src="https://github.com/dayaneLima/CalculadoraFreelancer0t/blob/master/Docs/Imgs/aula_05_subistituir_02.png" alt="Subistituir texto" width="100%">
 
Prontinho, a tipagem foi substituída, caso não queira fazer dessa forma, pode alterar método a método, onde há a tipagem Profissional trocando por TEntity.

### ProfissionalRepository

Vamos agora criar uma classe para implementar de nossa interface IProfissionalRepository. Ainda dentro da camada CalculadoraFreelancer.Infra.Data, 
dentro da pasta Repository crie uma classe chamada ProfissionalRepository.

```c#
 public class ProfissionalRepository
 {
 }
````

Agora ela devrá herdar da nossa classe de repositório base, a AzureRepository, que contém todas as operações base já implementadas. 

```c#
 public class ProfissionalRepository : AzureRepository
 {
 }
````

A AzureRepository também espera uma tipagem, pois a mesma é genérica, como estamos no repositório do Profissional, informaremos a classe que será do tipo Profissional.

```c#
 public class ProfissionalRepository : AzureRepository<Profissional>
 {
 }
````

Por fim temos que implementar da interface do repositório do profissional, a IProfissionalRepository, dessa forma:

```c#
 public class ProfissionalRepository : AzureRepository<Profissional>, IProfissionalRepository
  {
  }
````

Prontinho, nosso repositório do profissional está pronto.

### ProjetoRepository

Vamos fazer na ProjetoRepository o mesmo que foi feito na ProfissionalRepository. Ainda dentro da camada CalculadoraFreelancer.Infra.Data, 
dentro da pasta Repository crie uma classe chamada ProjetoRepository.

A classe deverá herdar do AzureRepository, informando a tipagem Projeto e deverá implementar da interface IProjetoRepository. Ficará dessa forma:

```c#
  public class ProjetoRepository : AzureRepository<Projeto>, IProjetoRepository
  {
  }
````

Pronto, nossa camada de Infra está corrigida.

## Camada Principal - CalculadoraFreelancer

Falta agora alterar na nossa camada principal. 


