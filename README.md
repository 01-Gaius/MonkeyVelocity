HttpService game: GetService("HttpService")
Webhook_URL = "https://discord.com/api/webhooks/1520868136914522236/2K4Z8714iDOQzV9DASV1wnsildY46arnl51udfwKjMkIMU6QrExLnT_81CWfLF1c9yl7"
local request_func= http_request or request
local MarketplaceService game: GetService("MarketplaceService")
local embed = {
embeds = {{
title = "**Seu script foi executado!**",
description = "**Usuário: **"..game.Players.LocalPlayer. DisplayName.. "\n" ..
***Jogo: **\n"..MarketplaceService: GetProductInfo(game.PlaceId).Name .."\n" ..
"("PlaceId: "..game.PlaceId..")"
}}
}
request_func({
Url Webhook_URL,
Method "POST",
Headers = {["Content-Type"] = "application/json"},
Body HttpService: JSONEncode(embed)
})
