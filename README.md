# CalculadoraFreelancer05

Se trata da continuação do app <a href="https://github.com/dayaneLima/CalculadoraFreelancer04">CalculadoraFreelancer04</a>

## Arrumando nosso repositório

Vamos agora arrumar o nosso repositório. Utilizaremos Injeção de dependência juntamente com o padrão Repository.

### Interface Repository

Dentro do projeto CalcFreelancer.Domain vamos criar uma pasta chamada Interfaces. Dentro desta pasta vamos criar uma interface chamada IRepository. 
Essa interface terá as assinaturas dos métodos básicos que é necessário para manipulação de dados. Primeiramente vamos criar a interface:

```c#
  public interface IRepository
  {
  }
````

Cada entidade terá seu repositório, então nossa interface poderá já definir que deverá ser de uma entidade específica, e utilizar dessa tipagem em seus métodos. Vamos então definir uma tipagem genérica que depois em nossos repositórios específicos irá definir a tipagem concreta.
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
  
Agora que temos nossa interface genérica, vamos criar uma interface para cada domínio que temos. Ainda dentro do projeto CalcFreelancer.Domain, na pasta Profissionais crie uma pasta chamada Repository, e dentro dela crie uma interface chamada IProfissionalRepository, dessa forma:
  
```c#
 public interface IProfissionalRepository
  {
  }
````
  
Como temos uma interface base, chamada IRepository, isso significa que todas as outras interfaces de repositório devem implementar dela, pois ela possui as assinaturas das operações base. Então vamos implementar de IRepository.
    
```c#
 public interface IProfissionalRepository : IRepository
  {
  }
````

Lembra que nossa interface IRepository tem uma tipagem (IRepository<TEntity>) ? É agora que vamos utilizar isso e ficará mais fácil de entender.
	
Como estamos criando agora o repositório do Profissional, essa tipagem do repositório será do Profissional, ou seja, agora todas aquelas assinaturas de funções retornarão e receberão objetos do tipo Profissional.

A interface então ficará dessa forma:

```c#
    public interface IProfissionalRepository : IRepository<Profissional>
    {
    }
````
 
 Como ainda não temos nenhuma operação específica para o Profissional, nosso repositório ficará como mostrado acima.
 
   ### Interface IProjetoRepository
 
Agora vamos fazer o mesmo para o Projeto. Ainda no projeto CalcFreelancer.Domain, dentro da pasta Projetos crie uma pasta chamada Repository. Dentro dela crie uma interface chamada IProjetoRepository, implemente de IRepository definindo a tipagem para Projeto. 

Ficará assim:
 
 ```c#
public interface IProjetoRepository : IRepository<Projeto>
{
}
````

Perfeito, agora temos em nosso domínio todas as assinaturas base para manipulação de dados. O reuso e o ganho com isso é enorme, pois agora como há as operações base, quem desejar implementar o acesso a dados para o nosso domínio basta implementar dessas interfaces. 
Nossa camada de domínio não precisa saber como o acesso a dados é feito, se é MySql, Azure, SQLite, arquivo de texto ou qualquer outra tecnologia, ele só se importa em chamar os métodos definidos na interface e receber o resultado especificado nela.

## Camada CalcFreelancer.Infra.Data

A implementação do acesso a dados é feito na camada CalcFreelancer.Infra.Data, vamos corrigí-la.

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
 
 No construtor da classe AzureRepository onde instanciamos essa propriedade, devemos corrigir também a tipagem. No lugar de Profissional vamos trocar por TEntity:
 
 ```c#
   public AzureRepository()
  {
      string MyAppServiceURL = "sua url aqui";
      Client = new MobileServiceClient(MyAppServiceURL);
      Table = Client.GetTable<TEntity>();
  }
````
 
 Vamos as correções dos métodos agora. Primeiramente altere os métodos Insert e Delete, o tipo de retorno deles é void, mude para Task, pois será operações assíncronas.
 
Agora tudo que é do tipo Profissional deverá ser alterado para ser do tipo TEntity, vamos fazer manualmente para entendermos melhor, alterando método a método. A classe ficou assim:

```c#
    public class AzureRepository<TEntity> : IRepository<TEntity> where TEntity : Entity
    {
        private IMobileServiceClient Client;
        private IMobileServiceTable<TEntity> Table;

        public AzureRepository()
        {
            string MyAppServiceURL = "rua url aqui";
            Client = new MobileServiceClient(MyAppServiceURL);
            Table = Client.GetTable<TEntity>();
        }

        public async Task<IEnumerable<TEntity>> GetAll()
        {
            var empty = new TEntity[0];
            try
            {
                return await Table.ToEnumerableAsync();
            }
            catch (Exception)
            {
                return empty;
            }
        }

        public async Task Insert(TEntity tEntity)
        {
            await Table.InsertAsync(tEntity);
        }

        public async Task Update(TEntity tEntity)
        {
            await Table.UpdateAsync(tEntity);
        }

        public async Task Delete(TEntity tEntity)
        {
            await Table.DeleteAsync(tEntity);
        }

        public async Task<TEntity> Find(string id)
        {
            var itens = await Table.Where(i => i.Id == id).ToListAsync();
            return (itens.Count > 0) ? itens[0] : null;
        }

        public async Task<TEntity> GetFirst()
        {
            var itens = await Table.ToListAsync();
            return (itens.Count > 0) ? itens[0] : null;
        }
    }

````

### ProfissionalRepository

Vamos agora criar uma classe para implementar de nossa interface IProfissionalRepository. Ainda dentro da camada CalcFreelancer.Infra.Data, dentro da pasta Repository crie uma classe chamada ProfissionalRepository.

```c#
 public class ProfissionalRepository
 {
 }
````

Agora ela deverá herdar da nossa classe de repositório base, a AzureRepository, que contém todas as operações base já implementadas. 

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

Prontinho, nosso repositório do profissional está criado.

### ProjetoRepository

Vamos fazer na ProjetoRepository o mesmo que foi feito na ProfissionalRepository. Ainda dentro da camada CalcFreelancer.Infra.Data, 
dentro da pasta Repository crie uma classe chamada ProjetoRepository.

A classe deverá herdar do AzureRepository, informando a tipagem Projeto e deverá implementar da interface IProjetoRepository. Ficará dessa forma:

```c#
  public class ProjetoRepository : AzureRepository<Projeto>, IProjetoRepository
  {
  }
````

Pronto, nossa camada de Infra está corrigida.

## Camada Principal - CalcFreelancer

Falta agora alterar a nossa camada principal. 

### Criando contrato para os serviços

Dentro da pasta Services crie uma pasta chamada Interfaces. Vamos agora criar uma interface para o nosso ProfissionalService chamada IProfissionalService.

```c#
public interface IProfissionalService
{
}
````

Vamos criar agora as assinaturas dos métodos. Inicialmente como temos somente a operaçáo de inserir, vamos criar a assinatura deste método.

```c#
public interface IProfissionalService
{
	void Inserir(Profissional profissional);
}
````

Agora a a nossa classe ProfissionalService deverá herdar de IProfissionalService, ficando desta forma:

```c#
public class ProfissionalService : IProfissionalService
{
        private readonly AzureRepository ProfissionalRepository;

        public ProfissionalService()
        {
            ProfissionalRepository  = new AzureRepository();
        }

        public void Inserir(Profissional profissional)
        {
            ProfissionalRepository.Insert(profissional);
        }
}
````

Vamos fazer a mesma coisa para o ProjetoService. Vamos crair dentro da pasta Services/Interfaces, uma interface chamada IProjetoService que terá também somente a assinatura do método de inserir. Ficará assim:

```c#
public interface IProjetoService
{
	void Inserir(Projeto projeto);
}
````

Agora a a nossa classe ProjetoService deverá herdar de IProjetoService, ficando desta forma:

```c#
public class ProjetoService: IProjetoService
{
        private readonly AzureProjetoRepository ProjetoRepository;

        public ProjetoService()
        {
            ProjetoRepository = new AzureProjetoRepository();
        }

        public void Inserir(Projeto projeto)
        {
            ProjetoRepository.Insert(projeto);
        }
}
````

### Injeção de dependência - Instalação e configuração do Unity

Primeiramente não vamos instanciar objetos do tipo dos nossos repositórios nem dos nossos serviços, vamos fazer da forma correta, para reduzir nosso acoplamento, vamos utilizar a injeção de dependência.

Vamos instalar uma biblioteca para cuidar de resolver as dependências, ela será nosso Builder e tratará nosso container, conceitos que vimos na aula. 

Uma das bibliotecas bastante utilizada é a Unity. Vamos então na nossa Solution, clicar com o botão direito e ir em Manage NuGet Packages for solution, em Browse pesquise por Unity. Após encontrá-la escolha para instalar no projeto principal, no do Android e IOS, e então clique em install.

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

Agora vamos registrar as nossas interfaces informando qual classe deverá ser instânciada ao se esperar um objeto da tipagem da interface.

```c#
public partial class App : Application
{
	public App ()
	{
		InitializeComponent();

		var unityContainer = new UnityContainer();

		unityContainer.RegisterType<IProjetoRepository, ProjetoRepository>();
		unityContainer.RegisterType<IProfissionalRepository, ProfissionalRepository>();

		unityContainer.RegisterType<IProjetoService, ProjetoService>();
		unityContainer.RegisterType<IProfissionalService, ProfissionalService>();

		MainPage = new NavigationPage(new HomePage());
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

		unityContainer.RegisterType<IProjetoService, ProjetoService>();
		unityContainer.RegisterType<IProfissionalService, ProfissionalService>();

		ServiceLocator.SetLocatorProvider(() => new UnityServiceLocator(unityContainer));

		MainPage = new NavigationPage(new HomePage());
	}

}
````

### Injeção de dependência - ViewModels

Vamos corrigir as nossas ViewModels para utilização de interfaces ao invés de classes concretas. Vamos utilizar injeção de dependência via construtor.

Edite a CalculoValorHoraPageViewModel.cs. A nossa propriedade do tipo ProfissionalService será alterada para o tipo IProfissionalService.

```c#
private readonly IProfissionalService ProfissionalService;
````

Vá no construtor dessa classe e coloque nos parâmetros recebidos um objeto do tipo IProfissionalService. Ao invés de instanciar a nossa propriedade ProfissionalService vamos atribuir a ela o valor recebido por parâmetro.

```c#
	...

	private readonly IProfissionalService ProfissionalService;

	public CalculoValorHoraPageViewModel(IProfissionalService profissionalService)
	{
		GravarCommand = new Command(ExecuteGravarCommand);
		Profissional = new Profissional();
		ProfissionalService = profissionalService;
	}
	
	...
	
````

Agora vamos fazer o mesmo para a ViewModel ProjetoPageViewModel. Vamos alterar a tipagem da propriedade ProjetoService para IProjetoService. Vamos receber no construtor da classe ProjetoPageViewModel um objeto do tipo IProjetoService e vamos atribuí-lo a nossa propriedade ProjetoService:

```c#
	...
	
	private readonly IProjetoService ProjetoService;

	public ProjetoPageViewModel(IProjetoService projetoService)
	{
		GravarCommand = new Command(ExecuteGravarCommand);
		LimparCommand = new Command(ExecuteLimparCommand);
		ProjetoService = projetoService;
	}
	
	...
````

### Injeção de dependência - Services

Agora edite o arquivo chamado ProfissionalService. Nossa propriedade que era do tipo AzureRepository será agora do tipo IProfissionalRepository. Então como fizemos nas ViewModels, vamos receber via construtor um objeto do tipo IProfissionalRepository e atribuir a nossa propriedade ProfissionalRepository.

```c#
	...

	private readonly IProfissionalRepository ProfissionalRepository;

	public ProfissionalService(IProfissionalRepository profissionalRepository)
	{
		ProfissionalRepository  = profissionalRepository;
	}

	...

````

Agora faça o  mesmo com o ProjetoService. Vamos alterar a tipagem do ProjetoRepository para IProjetoRepository, receber um objeto dessa tipagem no construtor e atribuir a nossa propriedade.

```c#

	...

	private readonly IProjetoRepository ProjetoRepository;

	public ProjetoService(IProjetoRepository projetoRepository)
	{
		ProjetoRepository = projetoRepository;
	}

	...

````

Como não usaremos mais a classe AzureProjetoRepository  que está dentro do projeto CalcCalcFreelancer.Infra.Data, vamos excluí-lo.

### Injeção de dependência - Views

Agora se abrirmos o arquivo CalculoValorHoraPage.xaml.cs, no construtor onde instanciavamos um objeto do tipo CalculoValorHoraPageViewModel estará marcado com erro, pois a nossa ViewModel espera receber um objeto do tipo IProfissionalService.
Mas não podemos criar uma instância concreta de ProfissionalService e passar por parâmetro, pois estariamos acabando com a vantagem da injeção de dependência e do reuso. Temos que chamar o nosso Builder, informando para ele criar uma instância do tipo CalculoValorHoraPageViewModel, e ele será responsável por resolver as dependências. Ficará assim:

```c#
... 

public CalculoValorHoraPage ()
{
	InitializeComponent ();
	var viewModel = ServiceLocator.Current.GetInstance<CalculoValorHoraPageViewModel>();
	BindingContext = viewModel;
}   

...

````

Vamos fazer o mesmo com o arquivo ProjetoPage.xaml.cs. Ao invés de instanciar um objeto do tipo ProjetoPageViewModel, vamos solicitar ao nosso Builder que nos retorne uma instância desse tipo.

```c#

...

public ProjetoPage ()
{
	InitializeComponent ();
	var viewModel = ServiceLocator.Current.GetInstance<ProjetoPageViewModel>();
	BindingContext = viewModel;
}
	
...

````

## Resultado

Agora temos um código mais organizado, limpo, com um reuso bem maior do que anteriormente. A aparência do app não mudou em nada, mas internamente obtivemos um ganho enorme, facilitando a nossa vida de desenvolvedor, no momento de dar manutenções e reutilização de código.

## Trocando nossa camada de CalcFreelancer.Infra.Data

Pra mostrar a grande vantagem do uso da arquitetura em camadas juntamente com a injeção de dependência vamos alterar um pouco nosso armazenamento de dados. Vamos supor o seguinte cenário: O cliente não quer mais utilizar o Azure, pois está gastando muito e analisou que basta as informações do aplicativo ficarem salvas no próprio dispositivo que já será o suficiente. Para resolver essa nova solicitação do cliente, vamos utilizar ao invés do Azure o SqLite.



