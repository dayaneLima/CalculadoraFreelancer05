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
 
 <img src="https://github.com/dayaneLima/CalculadoraFreelancer05/blob/master/Docs/Imgs/aula_05_subistituir_01.png" alt="Subistituir texto" width="100%">
 
 No primeiro campo digite o termpo que quer encontrar, no caso, Profissional, no segundo campo digite por qual palavra deseja substituir, no caso coloque TEntity. 
 Após clique para substituir todos, dessa forma:
 
 <img src="https://github.com/dayaneLima/CalculadoraFreelancer05/blob/master/Docs/Imgs/aula_05_subistituir_02.png" alt="Subistituir texto" width="100%">
 
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

Falta agora alterar a nossa camada principal. 

### Injeção de dependência - Instalação e configuração do Unity

Primeiramente não vamos instanciar objetos do tipo ProjetoRepository ou ProfissionalRepository, vamos fazer da forma correta, para reduzir nosso acoplamento, vamos utilizar a injeção de dependência.

Vamos instalar uma biblioteca para cuidar de resolver as dependências, ela será nosso Builder e tratará nosso container, conceitos que vimos na aula. 

Uma das bibliotecas bastante utilizada é a Unity. Vamos então na nossa solution, clicar com o botão direito e ir em Manage NuGet Packages for solution, em Browse pesquise por Unity. Após encontrá-la escolha para instalar no projeto principal, no do Android e IOS, e então clique em install.

 <img src="https://github.com/dayaneLima/CalculadoraFreelancer05/blob/master/Docs/Imgs/aula_05_install_unity_01.png" alt="Instalação Unity" width="100%">

Após a instalação vamos registrar nossos containers, ou seja, informar se for a interface IProjetoRepository, deverá entregar uma instância do tipo ProjetoRepository. Edite o arquivo chamado App.xaml.cs que se encontra em nosso projeto principal.

Em seu construtor, após a funcão InitializeComponent(), instâncie um objeto do tipo UnityContainer, dessa forma:

```c#
public partial class App : Application
{
	public App ()
	{
	      	InitializeComponent();

            	var unityContainer = new UnityContainer();

            	MainPage = new NavigationPage(new ProjetoPage());
	}
    
    ...
    
}
````

Agora vamos registrar as nossas interfaces informando de qual instância deve se criar a classe:

```c#
public partial class App : Application
{
      public App ()
      {
              InitializeComponent();

              var unityContainer = new UnityContainer();

              unityContainer.RegisterType<IProjetoRepository, ProjetoRepository>();
              unityContainer.RegisterType<IProfissionalRepository, ProfissionalRepository>();

              MainPage = new NavigationPage(new ProjetoPage());
      }
}
````

Vamos agora informar ao Builder que deverá utilizar esse container que acabamos de definir, dessa forma:

```c#
public partial class App : Application
{
      public App ()
      {
              InitializeComponent();

              var unityContainer = new UnityContainer();

              unityContainer.RegisterType<IProjetoRepository, ProjetoRepository>();
              unityContainer.RegisterType<IProfissionalRepository, ProfissionalRepository>();

              ServiceLocator.SetLocatorProvider(() => new UnityServiceLocator(unityContainer));

              MainPage = new NavigationPage(new ProjetoPage());
      }
}
````

### Injeção de dependência - ViewModels

Vamos corrigir as nossas ViewModels para utilização de interfaces ao invés de classes concretas. Vamos utilizar injeção de dependência via construtor.

Edite a CalculoValorHoraPageViewModel.cs. Crie uma propriedade para essa classe do tipo IProfissionalRepository:

```c#
public IProfissionalRepository ProfissionalRepository { get; }
````

Vá no construtor dessa classe e coloque nos parâmetros recebidos um objeto do tipo IProfissionalRepository, e o utilize para inicial a nossa propriedade criada anteriormente:

```c#
public IProfissionalRepository ProfissionalRepository { get; }

public CalculoValorHoraPageViewModel(IProfissionalRepository profissionalRepository)
{
    GravarCommand = new Command(ExecuteGravarCommand);
    Profissional = new Profissional();
    ProfissionalRepository = profissionalRepository;
}
````

Não foi mencionado anteriormente, mas o correto é que haja apenas um Profissional na tabela de Profissional do Azure, então ao abrir a tela vamos obter o profissional da base de dados do Azure.

Primeiramente remova a instanciação do objeto profissional do construtor e chame uma função chamada ObterProfissional que vamos criar no próximo passo:

```c#
public CalculoValorHoraPageViewModel(IProfissionalRepository profissionalRepository)
{
    GravarCommand = new Command(ExecuteGravarCommand);
    ProfissionalRepository = profissionalRepository;
    ObterProfissional();            
}
````

Agora vamos criar a função ObterProfissional, ela deverá chamar a função GetFirst do repository, caso retorne null é porque não foi encontrado nenhum profissional, então devemos instanciar nosso objeto Profissional, caso contrário, atribuimos o resultado da chamada GetFirst a nossa propriedade Profissional da ViewModel. 

```c#
private async void ObterProfissional()
{
    Profissional = await ProfissionalRepository.GetFirst() ?? new Profissional();
}
````

Caso tenha algo em nossa propriedade profissional devemos atualizar os valores da tela, vamos então criar uma funçào chamada AtualizaValoresTela, dessa forma:

```c#
private void AtualizaValoresTela()
{
    ValorGanhoMes = Profissional.ValorGanhoMes;
    HorasTrabalhadasPorDia = Profissional.HorasTrabalhadasPorDia;
    DiasTrabalhadosPorMes = Profissional.DiasTrabalhadosPorMes;
    DiasFeriasPorAno = Profissional.DiasFeriasPorAno;
    ValorDaHora = Profissional.ValorPorHora;
    DiasDoencaPorAno = Profissional.DiasDoencaPorAno;
}
````

Como estamos utilizando Bindable na propriedade Profissional, no set dessa propriedade podemos chamar a função AtualizaValoresTela, dessa forma:

```c#
private Profissional profissional;
public Profissional Profissional
{
    get { return profissional; }
    set
    {
	SetProperty(ref profissional, value);
	AtualizaValoresTela();
    }
}
````

Agora vamos na função ExecuteGravarCommand e chamar o nosso repository. Temos que verificar se há algo na propriedade Id do profissional, se há representa que o profissional já existe e deve então atualizá-lo, caso contrário deverá fazer uma inserção.

```c#
private async void ExecuteGravarCommand(object obj)
{
    Profissional.ValorGanhoMes = ValorGanhoMes;
    Profissional.HorasTrabalhadasPorDia = HorasTrabalhadasPorDia;
    Profissional.DiasTrabalhadosPorMes = DiasTrabalhadosPorMes;
    Profissional.DiasFeriasPorAno = DiasFeriasPorAno;
    Profissional.DiasDoencaPorAno = DiasDoencaPorAno;
    Profissional.ValorPorHora = ValorDaHora;

    if (!string.IsNullOrEmpty(profissional.Id))
	await ProfissionalRepository.Update(profissional);
    else
	await ProfissionalRepository.Insert(profissional);

    await App.Current.MainPage.DisplayAlert("Sucesso", "Valor por hora gravado!", "Ok");
}
````

### Injeção de dependência - Views

Agora se abrirmos o arquivo CalculoValorHoraPage.xaml.cs, no construtor onde instanciavamos um objeto do tipo CalculoValorHoraPageViewModel estará marcado com erro, pois a nossa ViewModel espera receber um objeto do tipo IProfissionalRepository.
Mas não podemos criar uma instância concreta de ProfissionalRepository e passar por parâmetro, pois estariamos acabando com a vantagem da injeção de dependência e do reuso. Temos que chamar o nosso Builder, informando para ele criar uma instância do tipo CalculoValorHoraPageViewModel, e ele será responsável por resolver as dependências. Ficará assim:

```c#
public CalculoValorHoraPage ()
{
	InitializeComponent ();
	var viewModel = ServiceLocator.Current.GetInstance<CalculoValorHoraPageViewModel>();
	BindingContext = viewModel;
}   
````

## Resultado

Agora temos um código mais organizado, limpo, com um reuso bem maior do que anteriormente. A aparência do app não mudou em nada, mas internamente obtivemos um ganho enorme, facilitando a nossa vida de desenvolvedor, no momento de dar manutenções e reutilização de código.
