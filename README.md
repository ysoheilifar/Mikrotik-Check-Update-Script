# Mikrotik Check for Update Script
Script check for update and send Telegram massage
1. The following permissions are required for the script to Сheck the RouterOS update: **read, write, policy, test**
```
System → Scripts → + → Name:CheckUpdate → Policy: read, write, test, policy
```
2. Script code for Telegram :
```
# Func: Telegram send message
:local TGSendMessage do={
    :local tgUrl "https://api.telegram.org/bot$Token/sendMessage?chat_id=$ChatID&text=$Text&parse_mode=html&disable_web_page_preview=True";
    /tool fetch http-method=get url=$tgUrl keep-result=no;
}

# Constants
:local TelegramBotToken "your-telegram-bot-token";
:local TelegramChatID "your-telegram-chatid";
:local DeviceName [/system identity get name];
:local TelegramMessageText "\F0\9F\9F\A2 <b> $DeviceName:</b>  ";


# Check Update
:local MyVar [/system package update check-for-updates as-value];
:local Chan ($MyVar -> "channel");
:local InstVer ($MyVar -> "installed-version");
:local LatVer ($MyVar -> "latest-version");


:if ($InstVer = $LatVer) do={
    :set TelegramMessageText  ($TelegramMessageText . "System is already up to date");
} else={
    
    :set TelegramMessageText  "$TelegramMessageText New version $LatVer is available! <a href=\"https://mikrotik.com/download/changelogs\">Changelogs</a>. [Installed version $InstVer, chanell $Chan].";

    $TGSendMessage Token=$TelegramBotToken ChatID=$TelegramChatID Text=$TelegramMessageText;
}

:log warning $TelegramMessageText;
```
3. Set your Telegram Bot Token and Telegram ChatID
 ```
 :local TelegramBotToken "your-telegram-bot-token"
 :local TelegramChatID "your-telegram-chatid";
 ```
 4. Script for Email:
 ```
 # Constants
:local date ([:pick [/system clock get date] 0 3] ."_". [:pick [/system clock get date] 4 6] ."_". [:pick [/system clock get date] 7 11]);
:local time ([:pick [/system clock get time] 0 2]."_".[:pick [/system clock get time] 3 5]);
:local sysname [/system identity get name];
:local toemail "your-email";
:local fromemail ($"sysname"."@your-domain");
:local emailserver "your-mail-server";
:local EmailMessageText "$sysname:";


# Check Update
:local MyVar [/system package update check-for-updates as-value];
:local Chan ($MyVar -> "channel");
:local InstVer ($MyVar -> "installed-version");
:local LatVer ($MyVar -> "latest-version");


:if ($InstVer = $LatVer) do={
    :set EmailMessageText  ($EmailMessageText . " System is already up to date");
} else={
    
    :set EmailMessageText  "$EmailMessageText New version $LatVer is available! [Installed version $InstVer, chanell $Chan].";

    /tool e-mail send to=$"toemail" from=$"fromemail" server=$"emailserver" port=587 subject="$sysname Mikrotik [Update Status]" body="Mikrotik $sysname Update Status on date: $date and time: $time \r\n $EmailMessageText";
}

:log warning $EmailMessageText;
 ```
 5. Set your Email informations:
 ```
:local toemail "your-email";
:local fromemail ($"sysname"."@your-domain");
:local emailserver "your-mail-server";
```
 6. Run the script and enjoy it :ok_hand:
