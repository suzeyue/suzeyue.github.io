---
title: Docker 1.12源码分析 -- Docker Client
date: 2017-08-24 19:56:12
tags:
  - docker
  - golang
---

在分析源码之前，我们先来思考几个问题：
1. Docker Client到底是干嘛的？
    Docker Client的任务就是接受用户的命令，并将之解析成Docker Server能够识别的形式，并发送至Docker Server。
2. Docker Client怎么使用的？
    我们先来看下Docker Client 的用法： 
```
Usage: docker [OPTIONS] COMMAND [arg...]
       docker [ --help | -v | --version ]
```
这里用法分为两种：第一种是 `docker --help | docker -v | docker --version` 显示docker的帮助或版本信息；另一种则是解析用户的命令，并发送给Docker Server。
首先是我们运行的命令 **docker** （docker是根命令），然后是可选参数 **[OPTIONS]** （options也就是我们常说的flag参数），然后是子命令 **command** ，最后是命令请求的参数 **arg** 。
3. 了解了Docker Client的作用之后，那么，Docker Client是怎么创建的？
4. flag、subcommand又是如何定义的？它们又是如何解析用户的命令？
5. 最后Docker Client又是如何将解析好的请求发送给Docker Server的呢？

带着上面的问题我们来查看Docker Client的源码，Docker Client的入口是cmd/docker/docker.go。

#### Docker命令中flag的解析
对于flag的定义以及用法，可以看我前文对mflag包的分析。
回到docker.go，首先是四个包级变量：
```
var (
	commonFlags = cliflags.InitCommonFlags()
	clientFlags = initClientFlags(commonFlags)
	flHelp      = flag.Bool([]string{"h", "-help"}, false, "Print usage")
	flVersion   = flag.Bool([]string{"v", "-version"}, false, "Print version information and quit")
)
```
**commonFlags：** commonFlags是client和daemon通用的flags。注册了debug模式、日志等级、tls相关、host等flag；定义了PostParse（）函数，PostParse主要设置日志等级以及tls验证的处理。
commonFlags里注册的flags：
```
cmd.BoolVar(&commonFlags.Debug, []string{"D", "-debug"}, false, "Enable debug mode")
cmd.StringVar(&commonFlags.LogLevel, []string{"l", "-log-level"}, "info", "Set the logging level")
cmd.BoolVar(&commonFlags.TLS, []string{"-tls"}, false, "Use TLS; implied by --tlsverify")
cmd.BoolVar(&commonFlags.TLSVerify, []string{"-tlsverify"}, dockerTLSVerify, "Use TLS and verify the remote")
cmd.StringVar(&tlsOptions.CAFile, []string{"-tlscacert"}, filepath.Join(dockerCertPath, DefaultCaFile), "Trust certs signed only by this CA")
cmd.StringVar(&tlsOptions.CertFile, []string{"-tlscert"}, filepath.Join(dockerCertPath, DefaultCertFile), "Path to TLS certificate file")
cmd.StringVar(&tlsOptions.KeyFile, []string{"-tlskey"}, filepath.Join(dockerCertPath, DefaultKeyFile), "Path to TLS key file")
cmd.Var(opts.NewNamedListOptsRef("hosts", &commonFlags.Hosts, opts.ValidateHost), []string{"H", "-host"}, "Daemon socket(s) to connect to")
```
**clientFlags：** clientFlags包含了commonFlags，注册了config，用于指定client的配置文件；在PostParse（）中设置config的目录、trustkey和是否开启debug模式。
clientFlags注册的flag：
```
client.StringVar(&clientFlags.ConfigDir, []string{"-config"}, cliconfig.ConfigDir(), "Location of client config files")
```
**flHelp：** help flag
**flVersion：** version flag
至此，docker client的所有flag都已注册完成。
因为刚刚实际使用了cliflags.CommonFlags、cliflags.ClientFlags、flag.CommandLine三种结构来存储注册的flags，因此需要将这些分散的flags合并：
```
flag.Merge(flag.CommandLine, clientFlags.FlagSet, commonFlags.FlagSet)
```
在之前对mflag包分析时就提过，flag在使用之前需要先解析：
```
flag.Parse()
```
所以在解析用户命令时，如果遇到 `-v | --version | -h | --help` ，就会将 **flHelp** 或 **flVersion** 指针指向的bool值设为真。因此下面的代码就会执行：
```
if *flVersion {
	showVersion()
	return
}
if *flHelp {
	// if global flag --help is present, regardless of what other options and commands there are,
	// just print the usage.
	flag.Usage()
	return
}
```
这里也就是输出版本信息或help信息之后直接退出。

####  Docker Client创建

```
clientCli := client.NewDockerCli(stdin, stdout, stderr, clientFlags)
```
这里根据之前解析的clientFlags来创建Docker Client。我们首先来看clientCli的结构信息：
```
type DockerCli struct {
	// initializing closure
	init func() error

	// configFile has the client configuration file
	configFile *configfile.ConfigFile
	// in holds the input stream and closer (io.ReadCloser) for the client.
	in io.ReadCloser
	// out holds the output stream (io.Writer) for the client.
	out io.Writer
	// err holds the error stream (io.Writer) for the client.
	err io.Writer
	// keyFile holds the key file as a string.
	keyFile string
	// client is the http client that performs all API operations
	client client.APIClient
	......
}
```
我们看看NewDockerCli是如何返回DockerCli实例的：
```
func NewDockerCli(in io.ReadCloser, out, err io.Writer, clientFlags *cliflags.ClientFlags) *DockerCli {
	cli := &DockerCli{
		in:      in,
		out:     out,
		err:     err,
		keyFile: clientFlags.Common.TrustKey,
	}

	cli.init = func() error {
		clientFlags.PostParse()
		cli.configFile = LoadDefaultConfigFile(err)

		client, err := NewAPIClientFromFlags(clientFlags, cli.configFile)
		if err != nil {
			return err
		}

		cli.client = client

		if cli.in != nil {
			cli.inFd, cli.isTerminalIn = term.GetFdInfo(cli.in)
		}
		if cli.out != nil {
			cli.outFd, cli.isTerminalOut = term.GetFdInfo(cli.out)
		}

		return nil
	}

	return cli
}
```
首先是对输入、输出和错误流的设置以及keyFile的设置，keyFile就是CommonFlags中的TrustKey。如果TrustKey没有设置，那么它的默认值就是 **~/.docker/key.json** （~/.docker为默认的配置文件目录，key.josn为默认trust key）。
然后设置init函数。在init函数中首先执行了传入的clientFlags的PostParse（），而clientFlagsPostParse（）会先调用CommonFlags的postParse（）函数。这样就会对之前注册的一系列flags进行验证：

1. 根据 **-l | --log-level** 设置日志等级（默认为info）
2. 如果设置 **--tlsverify** 为true，那么强制 **CommonFlags.TLS** 为true。
3. 根据是否进行tls验证来设置tls验证的信息，根据 **--tlscert** 和 **--tlskey** 来设置cert和key文件。
4. 根据 **--config** 设置客户端各种配置文件的目录，默认值为 **~/.docker** 。
5. 根据 **-D | --debug** 是否开启debug模式。

回到init函数中，接下来根据指定的本地配置文件来设置configFile。
根据传入的clientFlags和上文的configFile来创建APIClient。
```
func NewAPIClientFromFlags(clientFlags *cliflags.ClientFlags, configFile *configfile.ConfigFile) (client.APIClient, error) {
	host, err := getServerHost(clientFlags.Common.Hosts, clientFlags.Common.TLSOptions)
	if err != nil {
		return &client.Client{}, err
	}

	customHeaders := configFile.HTTPHeaders
	if customHeaders == nil {
		customHeaders = map[string]string{}
	}
	customHeaders["User-Agent"] = clientUserAgent()

	verStr := api.DefaultVersion
	if tmpStr := os.Getenv("DOCKER_API_VERSION"); tmpStr != "" {
		verStr = tmpStr
	}

	httpClient, err := newHTTPClient(host, clientFlags.Common.TLSOptions)
	if err != nil {
		return &client.Client{}, err
	}

	return client.NewClient(host, verStr, httpClient, customHeaders)
}
```
首先是host的获取，这里host也就是Docker Server提供所要监听的对象。host首先是通过 **-H | --host=[]** 来指定的，如果没有指定，那么会从环境变量 **DOCKER_HOST** 中获取。如果这里也没有，同时没有指定tls认证的话，那么就会设置为默认值： **DefaultHost** ，其值为 **unix:///var/run/docker.sock** ；没有host却指定tls验证，那么就会设置成带tls的默认值： **DefaultTLSHost** ，其值为 **tcp://localhost:2376**  。
还有一个重要的就是httpClient的创建。如果tlsOptions为nil，那么http.Client为nil（如果为nil，APIClient会配置默认的transport）；否则根据tlsOptions创建http.Client。
这样APIClient创建好了，DockerCli也就创建好了。

#### CobraAdaptor的创建
```
cobraAdaptor := cobraadaptor.NewCobraAdaptor(clientFlags)
```
CobraAdaptor是一个用来支持 **spf13/cobra** 的适配器。
这里先介绍一下spf13/cobra。Cobra主要有两个作用，这里我们只关心第一个。当Cobra作为一个库时，提供了简单的接口来创建命令行接口（类似git和go这样的工具）。Cobra工作原理是先创建一系列的Command然后将它们组织成树结构，最上层是root command，在这里就是 **docker** ，然后每个Command都可以定义子Command。当一个Command执行时，会根据它的flag来运行具体的逻辑。Cobra： [github.com/spf13/cobra](https://github.com/spf13/cobra) 
CobraAdaptor结构：
```
type CobraAdaptor struct {
	rootCmd   *cobra.Command
	dockerCli *client.DockerCli
}
```
**rootCmd：** 根命令，在这里就是 **docker** 。
rootCmd的定义：
```
var rootCmd = &cobra.Command{
		Use:           "docker [OPTIONS]",
		Short:         "A self-sufficient runtime for containers",
		SilenceUsage:  true,
		SilenceErrors: true,
	}
```
同时还添加了很多子命令：
```
rootCmd.AddCommand(
		node.NewNodeCommand(dockerCli),
		service.NewServiceCommand(dockerCli),
		container.NewCommitCommand(dockerCli),
		container.NewCopyCommand(dockerCli),
		.......
)		
```
**dockerCli：** 就是上面介绍的Docker Client。

#### Docker Client的运行
Docker Client的运行是通过Cli的Run方法。Cli是什么？Cli是命令行接口。我们先来看Cli的结构：
```
type Cli struct {
	Stderr   io.Writer
	handlers []Handler
	Usage    func()
}
```
这里我们需要关注 **handlers** ，它是 **Handler** 型的slice。那 **Handler** 又是什么？
```
type Handler interface {
	Command(name string) func(...string) error
}
```
Handler接口规定了 **Command** 方法：根据指定的名称返回一个函数。
我们再来看看Cli的创建：
```
c := cli.New(clientCli, NewDaemonProxy(), cobraAdaptor)

//此函数在docker/cli/cli.go中
func New(handlers ...Handler) *Cli {
	// make the generic Cli object the first cli handler
	// in order to handle `docker help` appropriately
	cli := new(Cli)
	cli.handlers = append([]Handler{cli}, handlers...)
	return cli
}
```
我们可以看到New函数的参数是Handler类型的可变参数，具体做法是将这些Handler都添加到Cli.handlers中（包括自己）。
在创建时传入的参数为：clientCli、NewDaemonProxy()、 cobraAdaptor，这就说明它们都实现了Command方法。
- clientCli的类型是DockerCli，DockerCli的Command方法是通过预先定义好的 **map[string]func(...string)** ，然后通过取name对应的方法，其中cli.CmdExec等就是对应的处理函数了。
```
func (cli *DockerCli) Command(name string) func(...string) error {
	return map[string]func(...string) error{
		"exec":    cli.CmdExec,
		"info":    cli.CmdInfo,
		"inspect": cli.CmdInspect,
		"update":  cli.CmdUpdate,
	}[name]
}
```
- NewDaemonProxy()返回的是DaemonProxy类型，它的Command方法和上面类似：
```
func (p DaemonProxy) Command(name string) func(...string) error {
	return map[string]func(...string) error{
		"daemon": p.CmdDaemon,
	}[name]
}
```
- CobraAdaptor的Command方法稍微复杂一点，首先是range自己的子命令，如果名称相同，就会返回处理函数。
```
func (c CobraAdaptor) Command(name string) func(...string) error {
	for _, cmd := range c.rootCmd.Commands() {
		if cmd.Name() == name {
			return func(args ...string) error {
				return c.run(name, args)
			}
		}
	}
	return nil
}
```

最后，我们来看如何Cli是如何运行的：
```
if err := c.Run(flag.Args()...); err != nil {
		......
	}
```
运行是调用Cli的Run方法，参数是flag.Args()。flag.Args()的返回值就是命令行被解析之后，flag之后的参数。举个例子：
`docker -H=192.168.1.1 ps -a` ，这条命令经过解析之后192.168.1.1这个地址就会被添加到CommonFlags.Host中，而后面的 `ps -a` 则会被添加到FlagSet.args中，而flag.Args()返回的正是这个args（在例子中也就是 ["ps", "-a"]）。
具体的Run方法：
```
func (cli *Cli) Run(args ...string) error {
	if len(args) > 1 {
		command, err := cli.command(args[:2]...)
		if err == nil {
			return command(args[2:]...)
		}
		if err != errCommandNotFound {
			return err
		}
	}
	if len(args) > 0 {
		command, err := cli.command(args[0])
		if err != nil {
			if err == errCommandNotFound {
				cli.noSuchCommand(args[0])
				return nil
			}
			return err
		}
		return command(args[1:]...)
	}
	return cli.CmdHelp()
}
```
因为参数是 `docker [OPTIONS] COMMAND [arg...]` 中的 `COMMAND [arg...]` ，因此可以分为三种情况：
1. `COMMAND [arg...]` 都存在；
2.  只有`COMMAND` ；
3.  什么都没有（直接返回帮助信息）。

这里先判断参数是否大于1，是因为有些 `COMMAND` 可能会有subCommamd。比如： `docker swarm init` 。这里的init其实swarm的子命令，而不是swarm的参数。
`command, err := cli.command(args[:2]...)` 这里需要注意两个command不是一个概念，前面是后面的返回值。
我们来看一下后面的那个command方法：
```
func (cli *Cli) command(args ...string) (func(...string) error, error) {
	for _, c := range cli.handlers {
		if c == nil {
			continue
		}
		if cmd := c.Command(strings.Join(args, " ")); cmd != nil {
			if ci, ok := c.(Initializer); ok {
				if err := ci.Initialize(); err != nil {
					return nil, err
				}
			}
			return cmd, nil
		}
	}
	return nil, errCommandNotFound
}
```
其实就是range我们传入的Handlers，调用每个Handler的Command方法来找到与名称对应的函数，如果同样实现了Initializer的接口则运行Initialize方法，最后将函数返回，赋值给command。
最后一步，则是运行这个返回的函数，在这个函数中会向Docker Server发起HTTP请求。

