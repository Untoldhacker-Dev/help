# BJS Security

BJS is very powerful and flexible. But with this. But simply make the code **vulnerable**.

{% hint style="danger" %}
Please read this article carefully

Especially if you work with payments, user balances, sell goods through a bot
{% endhint %}

## Any user can execute any command

Vulnerable command `/setBalance`:

```javascript
// admin can add 100$ to users's balance by it telegramid

tgID = params

let res = Libs.ResourcesLib.anotherUserRes("money", tgID);
res.add(100)
Bot.sendMessage("Added 100$ for user");
```

{% hint style="danger" %}
Any user can run /setBalance \[telegramid\]
{% endhint %}

So need check execute this command only for admin:

**Add this command to group:**

![](../.gitbook/assets/image%20%2830%29.png)

**Or you can check admin in BJS:**

```javascript
if(user.telegramid!=ADMIN_TELEGRAM_ID){
  return // exit from BJS
}

// ONLY admin can add 100$ to users's balance by it telegramid

tgID = params

let res = Libs.ResourcesLib.anotherUserRes("money", tgID);
res.add(100)
Bot.sendMessage("Added 100$ for user");
```



## Any user can execute any "SECRET" command

For example, you have command `/payment` \(have "Wait for answer"\) with execute other "secret" command `/setBalance` :

```javascript
// command /payment

// user provide oneTime password. If password is valid - add bonus 100$

var oneTimePassword = User.getProperty("oneTimePassword");

if(!oneTimePassword){
  return // we have not oneTime password now
}

if(oneTimePassword=="already taked"){
  // if taked already - exit
  return
}

if(oneTimePassword!=message){
  // user do not know oneTime password
  Bot.sendMessager("Error. Password is wrong")
}

if(oneTimePassword==message){
  // user know oneTime password!
  // make it "already taked"
  User.setProperty("oneTimePassword", "already taked", "string")
  // run "secret" command
  Bot.runCommand("/setBalance");
  Bot.sendMessage("Thank you for payment!");
}
```

"Secret" command `/setBalance`

```javascript
let res = Libs.ResourcesLib.userRes("money", tgID);
res.add(100)
Bot.sendMessage("Added 100$ for you");
```

**So user must:**

* run /payment command
* type secret one time password
* after it - "secret" command "/setBalance" will be runned

**Vulnerability: hacker can run /setBalance only and get bonus immediately**

Need to checking that command `/setBalance` was runned only by command `/payment`

one of the methods - pass secret on run command as params:

command `/payment`

```javascript
...
// part of code for /payment

if(oneTimePassword==message){
  ...
  var secret = "GJHURFVJLHF" // use own secret. You can store it in property
  Bot.runCommand("/setBalance");
  Bot.sendMessage("Thank you for payment!");
}
```

{% hint style="danger" %}
Do not use "GJHURFVJLHF" secret! 

It is not secret world already
{% endhint %}

command `/setBalance`

```javascript
if(params=="GJHURFVJLHF"){
   let res = Libs.ResourcesLib.userRes("money", tgID);
   res.add(100)
   Bot.sendMessage("Added 100$ for you");
}else{
   Bot.sendMessage("You are hacker!")
}
```



## Recommendations

### Do not share your bot token, BB API Key

Bot token and BB API Key - are is very vulnerability data. Do not share theys anywhere!



### Do not use default command names "/onIncome", "/onTransaction" for important commands

Hacker can brute force such command names and try to execute it



## Do not use any non official libs now.

{% hint style="danger" %}
Do not use any non official libs now. 

* Any lib can run command with options.
* Any libs can read properties \(and read your API Keys from other lib\)

We have not way to protect this now. Just **not use NON official libs** with CP lib. Well, that now there are no such libraries
{% endhint %}
