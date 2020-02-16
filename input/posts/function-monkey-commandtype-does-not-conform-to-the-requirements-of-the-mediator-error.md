Title: Function Monkey Command type MyTypeHandler does not conform to the requirements of the mediator Error
Published: 2020-02-16
Tags: 
    - Function Monkey
    - Azure Functions
    - Net
    - Azure
---


I've lost an hour this morning to what turned out to be a dumb error on my part, so I'm posting this just in case anyone hits this issue in future.

I'm using [James Randall's](https://twitter.com/AzureTrenches)  [Function Monkey Library](https://github.com/JamesRandall/FunctionMonkey) for .NET Azure Functions which uses a [Mediation framework](https://github.com/JamesRandall/AzureFromTheTrenches.Commanding) by the same author to map incoming command objects to handlers.

Here's the error I was getting.

```
  FunctionMonkey.Compiler.targets(29, 5): Command type EditLocationHandler does not conform to the requirements of the mediator. 
  
  Commands must implement ICommand or ICommand<T>

```

Here's my EditLocationHandler

```   
    public class EditLocationHandler : ICommandHandler<EditLocationCommand, Location>
    {
        private readonly ILocationRepository _locationRepository;
        private readonly ILogger<EditLocationHandler> _logger;

        public EditLocationHandler(ILocationRepository locationRepository,
            ILogger<EditLocationHandler> logger)
        {
            _locationRepository = locationRepository;
            _logger = logger;
        }
        
        public async Task<Location> ExecuteAsync(EditLocationCommand command, Location previousResult)
        {
            //snipped For brevity
        }
    }
```

And my EditLocationCommand

```
 public class EditLocationCommand : ICommand<Location>
    {
        public Guid Id { get; set; }
        public string Name { get; set; }
        public double Latitude { get; set; }
        public double Longitude { get; set; }
        public string User { get; set; }
        public string Notes { get; set; }
    }
```

You can probably see why I was stumped, the handler and command implement the very same interfaces the error was telling me I should implement. 

After going down the usual rabbit hole which usually ends in me questioning why I'm even a software developer in the first place, I eventually realised the problem was in the Function Configuration file where I'd registered the commands.


```
  .HttpFunction<AddLocationCommand>(HttpMethod.Post)
  .HttpFunction<EditLocationHandler>(HttpMethod.Put)

```

HttpFunction expects something of type `TCommand`, *not* `TCommandHandler`.

Yep, I'm an idiot.
