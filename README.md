# Telegram-Bot-API-Client
This is a .Net Standard C# client for communication with the Telegram Bot API.

You will find Telegram bot api documentation at:
* https://core.telegram.org/bots
* https://core.telegram.org/bots/api

## Supported telegram bot api endpoints:
* '/sendMessage' to send message to a chat by given chat_id
* '/getUpdates' get updates (messages) sent to the bot
* '/setWebhook' updates what webhook settings should be used.

## Note!
If '/setWebhook' is used you cannot receive updates through polling '/getUpdates' due to limitations in telegram bot api. You will also have to use a HTTPS connection for your webhook.
Read more at: https://core.telegram.org/bots/api#setwebhook

## Todo
* Implement more telegram bot api endpoints
* Nuget package
* Add tests

## Example projects in solution
* One example of a .net core console application polling updates from bot api through '/getUpdates'
* One example of a .net core web application setting webhook that receives updates and reply to the sender.

**Example code for setting up webhook, see example project in solution**
```cs
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    // Add framework services.
    services.AddMvc();

    // Setup telegram client
    var accessToken = Configuration["Settings:accessToken"];
    TelegramClient telegramClient = new TelegramClient(accessToken);
    
    // Set up webhook
    string webhookUrl = Configuration["Settings:webhookUrl"];
    int maxConnections = int.Parse(Configuration["Settings:maxConnections"]);
    UpdateType[] allowedUpdates = { UpdateType.MessageUpdate };

    telegramClient.SetWebhook(webhookUrl, maxConnections, allowedUpdates);

    services.AddScoped<ITelegramClient>(client => telegramClient);
}
```

**Example code receving webhook update and sending a replying message, see example project in solution**
```cs
[Route("api/[controller]")]
public class TelegramController : Controller
{
    ITelegramClient _telegramClient;
    public TelegramController(ITelegramClient telegramClient) {
        _telegramClient = telegramClient;
    }

    // GET api/telegram/update/{token}
    [HttpPost("update/{token}")]
    public void Update([FromRoute]string token, [FromBody]Update update)
    {
        if (token != _telegramClient.GetConfiguration().AuthenticationToken)
            return;

        if(update != null && update.Message != null)
            _telegramClient.SendMessage(update.Message.Chat.Id, $"I received your message: \"{update.Message.Text}\"");
        
        return;
    }
    
}
```

