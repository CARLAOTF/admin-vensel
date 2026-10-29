local player=game:GetService("Players").LocalPlayer
if not player then warn("Vercel: entre no jogo primeiro");return end
local parent=player:FindFirstChildOfClass("PlayerGui")
if not parent then warn("Vercel: PlayerGui indisponível");return end
local gui=Instance.new("ScreenGui")
gui.Name="VercelCompilacao"
gui.ResetOnSpawn=false
gui.DisplayOrder=1000
local text=Instance.new("TextLabel")
text.Size=UDim2.new(0,280,0,100)
text.Position=UDim2.new(0.5,-140,0.5,-50)
text.BackgroundColor3=Color3.new(0,0,0)
text.TextColor3=Color3.new(1,1,1)
text.TextSize=17
text.TextWrapped=true
text.Text="Vercel: verificando o painel..."
text.Parent=gui
gui.Parent=parent
local function compilePanel(compiler,source)
 if type(compiler)~="function" then return nil,"Este ambiente não oferece loadstring." end
 local ok,fn,err=pcall(compiler,source,"VercelAdmin")
 if not ok or type(fn)~="function" then
  local detail=tostring(ok and err or fn)
  local line=detail:match(":(%d+):")
  return nil,"O executador recusou o painel antes de abrir"..(line and (" (linha "..line..").") or ".")
 end
 return fn
end
local panelSource=[====[local startupStage="início"
local startupFinished=false
local startupGui
local startupText
local function startupResult(ok,stage)
 if ok then return "Painel pronto",true end
 return "O painel parou em: "..tostring(stage)..". Envie uma foto deste aviso.",false
end
local diagnosticOk=pcall(function()
 local player=game:GetService("Players").LocalPlayer
 if not player then error("cliente ausente") end
 local parent=player:FindFirstChildOfClass("PlayerGui")
 if not parent then error("PlayerGui ausente") end
 startupGui=Instance.new("ScreenGui")
 startupGui.Name="VercelAdminDiagnostico"
 startupGui.ResetOnSpawn=false
 startupGui.DisplayOrder=999
 startupText=Instance.new("TextLabel")
 startupText.Size=UDim2.new(0,280,0,100)
 startupText.Position=UDim2.new(0.5,-140,0.5,-50)
 startupText.BackgroundColor3=Color3.new(0,0,0)
 startupText.TextColor3=Color3.new(1,1,1)
 startupText.TextSize=17
 startupText.TextWrapped=true
 startupText.Text="Vercel: abrindo painel..."
 startupText.Parent=startupGui
 startupGui.Parent=parent
end)
if not diagnosticOk then warn("VERCEL: não foi possível abrir o aviso inicial");return end
local function startupCheckpoint(stage)
 startupStage=stage
 if startupText and startupText.Parent then startupText.Text="Vercel: "..stage end
end
local LocationRules=(function()
local Rules={}
Rules.ownerId=157973654
Rules.owner=function(sender) return sender~=nil and tonumber(sender.UserId)==Rules.ownerId end
Rules.farm=function(name,plus,coin,grants)
 if name~="coin" and name~="candy" then return false end
 return (grants and grants[name]==true) or (plus==true and (name~="coin" or coin==true))
end
Rules.hideTags=function(dm,enabled,isSelf) return dm==true and enabled==true and not isSelf end
Rules.groupCreate=function(plus) return plus==true end
Rules.groupLaunch=function(plus) return plus==true end
Rules.groupRead=function(members,id) return type(members)=="table" and members[tostring(id)]==true end
Rules.cframe=function(components)
 if type(components)~="table" or #components~=12 then return nil end
 local out={}
 for i,n in ipairs(components) do
  if type(n)~="number" or n~=n or math.abs(n)==math.huge then return nil end
  out[i]=string.format("%.17g",n)
 end
 return "CFrame.new("..table.concat(out,", ")..")"
end
return Rules
end)()
local function runPanel()
startupCheckpoint("serviços Roblox")
local API_URL   = "https://vercel-api-ten-phi.vercel.app"
local ADMIN_KEY = ""
local PREFIX    = "!"

local Players = game:GetService("Players")
local HttpService = game:GetService("HttpService")
local TeleportService = game:GetService("TeleportService")
local tpFailConn = nil
local TextChatService = game:GetService("TextChatService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer
if not player then error("Abra o painel no cliente Roblox, depois de entrar no jogo.") end

startupCheckpoint("ambiente e painel anterior")
local env = (type(getgenv)=="function" and getgenv()) or _G
if type(env.VHUB_ADMIN_SHUTDOWN) == "function" then pcall(env.VHUB_ADMIN_SHUTDOWN) end
if type(env.VHUB_ADMIN_LOGIN_CLOSE)=="function" then pcall(env.VHUB_ADMIN_LOGIN_CLOSE) end
local alive = true
local function guiParent()
 local verifiedParent=player:FindFirstChildOfClass("PlayerGui")
 if verifiedParent then return verifiedParent end
 if type(gethui)=="function" then
  local ok,parent=pcall(gethui)
  if ok and typeof(parent)=="Instance" then return parent end
 end
 local parent=player:FindFirstChildOfClass("PlayerGui") or player:WaitForChild("PlayerGui",10)
 if not parent then error("PlayerGui indisponível; entre no jogo e tente novamente.") end
 return parent
end
startupCheckpoint("destino da janela")
local parentGui=guiParent()
local existingKey=env.VERCEL_ADMIN_KEY or env.VHUB_ADMIN_KEY
if type(existingKey)=="string" then ADMIN_KEY=existingKey end

local httpRequest = (type(syn) == "table" and syn.request) or (type(http) == "table" and http.request) or http_request or request

local uiReady=false
local connected=false
local connecting=false
local reportApiError
local function requestWithTimeout(options)
 local done=false
 local result
 local succeeded=false
 task.spawn(function()
  succeeded,result=pcall(httpRequest,options)
  done=true
 end)
 local deadline=os.clock()+15
 while alive and not done and os.clock()<deadline do task.wait(0.05) end
 if not done then return nil,"a API não respondeu em 15 segundos" end
 if not succeeded or type(result)~="table" then return nil,"falha de conexão" end
 return result
end
local function apiInner(path,body,verifying)
 if not uiReady then return nil,"painel ainda está abrindo" end
 if not connected and not verifying then return nil,"conecte o painel primeiro" end
 if ADMIN_KEY=="" then return nil,"digite a senha de administrador" end
 if type(httpRequest)~="function" then return nil,"sem http" end
 local encodedOk,encoded=pcall(HttpService.JSONEncode,HttpService,body)
 if not encodedOk then return nil,"pedido inválido" end
 local res,requestError=requestWithTimeout({Url=API_URL..path,Method="POST",Headers={["Content-Type"]="application/json",["x-admin-key"]=ADMIN_KEY},Body=encoded})
 if not res then return nil,requestError end
 local code=tonumber(res.StatusCode or res.Status)
 if code==401 then return nil,"senha recusada pela API" end
 if code==403 then return nil,"acesso negado: confira a senha e o proprietário configurado na API" end
 if code==404 then return nil,"endereço de administrador não encontrado na API" end
 if res.Success==false or (code and (code<200 or code>=300)) then return nil,"HTTP "..tostring(code) end
 local data=res.Body
 if type(data)~="table" then local decoded;decoded,data=pcall(HttpService.JSONDecode,HttpService,tostring(data or ""));if not decoded then return nil,"resposta inválida" end end
 if type(data)~="table" or data.success~=true then return nil,type(data)=="table" and data.error or "resposta incompleta" end
 return data
end
local function api(path,body,verifying)
 local data,err=apiInner(path,body,verifying)
 if not data and reportApiError then reportApiError(err) end
 return data,err
end
local function sendCommand(name, args, targets, scope)
	return api("/admin/command", {
		issuerId = tostring(player.UserId), jobId = game.JobId,
		name = name, args = args or {}, targets = targets, scope = scope or "server"
	})
end





local AdminCommands = {
	{name = "kill",     label = "💀 Matar"},
	{name = "me",       label = "🧲 Trazer até mim"},
	{name = "freeze",   label = "🧊 Congelar [seg]"},
	{name = "unfreeze", label = "🔥 Descongelar"},
	{name = "fling",    label = "🚀 Arremessar"},
	{name = "flingfar", label = "☄️ Arremessar longe"},
	{name = "trip",     label = "🍌 Derrubar [seg]"},
	{name = "spin",     label = "🌀 Girar [seg]"},
	{name = "jump",     label = "⬆️ Pular"},
	{name = "sit",      label = "🪑 Sentar"},
	{name = "speed",    label = "⚡ Velocidade [v] [seg]"},
	{name = "blind",    label = "🙈 Cegar [seg]"},
	{name = "shake",    label = "📳 Tremer tela [seg]"},
	{name = "dance",    label = "💃 Dançar [id] [seg]"},
	{name = "say",      label = "💬 Falar (texto)", joinArgs = true},
	{name = "rejoin",   label = "🔁 Rejoin"},
	{name = "reload",   label = "♻️ Recarregar script"},
	{name = "kick",     label = "👢 Kickar [motivo]", joinArgs = true},
}


local byName = {}
for _, c in ipairs(AdminCommands) do byName[c.name] = c end
byName.announce = {name = "announce", joinArgs = true}


local C = {
 bg = Color3.fromRGB(13,16,23), panel = Color3.fromRGB(20,24,32), field = Color3.fromRGB(24,30,40),
 btn = Color3.fromRGB(28,35,45), btnHover = Color3.fromRGB(37,49,61), accent = Color3.fromRGB(34,211,238),
 text = Color3.fromRGB(230,247,251), sub = Color3.fromRGB(124,142,158), good = Color3.fromRGB(52,211,153),
 bad = Color3.fromRGB(248,113,113), stroke = Color3.fromRGB(43,65,77), selected = Color3.fromRGB(22,57,66),
 ink = Color3.fromRGB(10,16,20), plus = Color3.fromRGB(168,85,247),
}
local fontBody,fontBold
pcall(function()
 fontBody=Font.new("rbxasset://fonts/families/Nunito.json",Enum.FontWeight.Normal)
 fontBold=Font.new("rbxasset://fonts/families/Nunito.json",Enum.FontWeight.Bold)
end)
local function new(class, props, parent)
	startupCheckpoint("criar "..class)
	local o = Instance.new(class)
	for k, v in pairs(props or {}) do startupCheckpoint(class.." / "..k);o[k] = v end
	if o:IsA("TextLabel") or o:IsA("TextButton") or o:IsA("TextBox") then
        local face=(o.Font == Enum.Font.GothamBold or o.Font == Enum.Font.GothamMedium) and fontBold or fontBody
        if face then pcall(function() o.FontFace=face end) end
    end
	o.Parent = parent
	return o
end
local function corner(o, r) new("UICorner", {CornerRadius = UDim.new(0, r or 6)}, o) end
local function stroke(o, c) new("UIStroke", {Color = c or C.stroke, Transparency = 0.45, Thickness = 1, ApplyStrokeMode = Enum.ApplyStrokeMode.Border}, o) end
local function button(parent, text, order, size)
	local b = new("TextButton", {Size = size or UDim2.new(1, 0, 0, 28), BackgroundColor3 = C.btn, TextColor3 = C.text, Font = Enum.Font.GothamMedium, TextSize = 13, Text = text, AutoButtonColor = false, LayoutOrder = order or 0}, parent)
	corner(b)
	b.MouseEnter:Connect(function() if b:GetAttribute("on") ~= true then b.BackgroundColor3 = C.btnHover end end)
	b.MouseLeave:Connect(function() if b:GetAttribute("on") ~= true then b.BackgroundColor3 = C.btn end end)
	return b
end
local function setOn(b, on)
	b:SetAttribute("on", on)
	b.BackgroundColor3 = on and C.accent or C.btn
	b.TextColor3 = on and C.ink or C.text
end
local function label(parent, text, order, size, color, font)
	return new("TextLabel", {Size = UDim2.new(1, 0, 0, size or 16), BackgroundTransparency = 1, Text = text, TextColor3 = color or C.sub, Font = font or Enum.Font.GothamBold, TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left, LayoutOrder = order or 0}, parent)
end
local function box(parent, placeholder, order, height, multi)
	local b = new("TextBox", {Size = UDim2.new(1, 0, 0, height or 28), BackgroundColor3 = C.field, TextColor3 = C.text, PlaceholderText = placeholder, PlaceholderColor3 = C.sub, Text = "", Font = Enum.Font.Gotham, TextSize = 13, ClearTextOnFocus = false, TextXAlignment = Enum.TextXAlignment.Left, TextYAlignment = multi and Enum.TextYAlignment.Top or Enum.TextYAlignment.Center, MultiLine = multi or false, TextWrapped = multi or false, LayoutOrder = order or 0}, parent)
	corner(b); new("UIPadding", {PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, multi and 6 or 0)}, b)
	return b
end
local function avatarUrl(userId) return "rbxthumb://type=AvatarHeadShot&id=" .. tostring(userId) .. "&w=48&h=48" end
local function fmtDur(sec)
	sec = tonumber(sec) or 0
	local h = math.floor(sec / 3600); local m = math.floor((sec % 3600) / 60)
	if h > 0 then return string.format("%dh %02dm", h, m) end
	return string.format("%dm", m)
end


local gui = new("ScreenGui", {Name = "VercelAdmin", ResetOnSpawn = false, ZIndexBehavior = Enum.ZIndexBehavior.Sibling, DisplayOrder = 100}, parentGui)

local frame = new("Frame", {Size = UDim2.fromOffset(720,500), Position = UDim2.new(0.5,-360,0.5,-250), BackgroundColor3 = C.bg, Active = true, Draggable = true}, gui)
corner(frame, 10); stroke(frame, C.accent)
local uiScale = new("UIScale", {Scale = 1}, frame)
local function fitWindow()
 local camera = workspace.CurrentCamera
 if not camera then return end
 local v = camera.ViewportSize
 uiScale.Scale = math.min(1, math.max(0.35, math.min((v.X-24)/720,(v.Y-24)/500)))
 frame.Position = UDim2.new(0.5,-360*uiScale.Scale,0.5,-250*uiScale.Scale)
end
fitWindow()
local cameraConn
if workspace.CurrentCamera then cameraConn = workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(fitWindow) end
local titleBar = new("Frame", {Size = UDim2.new(1,0,0,42), BackgroundTransparency = 1}, frame)
local dot = new("Frame", {Size = UDim2.fromOffset(6,6), Position = UDim2.fromOffset(14,18), BackgroundColor3 = C.good}, titleBar); corner(dot,6)
new("TextLabel", {Size = UDim2.fromOffset(160,42), Position = UDim2.fromOffset(28,0), BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 16, TextColor3 = C.accent, TextXAlignment = Enum.TextXAlignment.Left, Text = "Vercel 4.3 / ADM"}, titleBar)
new("TextLabel", {Size = UDim2.fromOffset(220,42), Position = UDim2.new(1,-264,0,0), BackgroundTransparency = 1, TextSize = 12, TextColor3 = C.sub, TextXAlignment = Enum.TextXAlignment.Right, TextTruncate = Enum.TextTruncate.AtEnd, Text = "@" .. player.Name}, titleBar)
local minBtn = button(titleBar,"×",0,UDim2.fromOffset(28,28)); minBtn.Position = UDim2.new(1,-38,0,7); minBtn.TextSize = 22
local reopen = button(gui,"V",0,UDim2.fromOffset(34,34)); reopen.Position = UDim2.new(0,12,0.5,-17); reopen.Visible = false; reopen.Active = true; reopen.Draggable = true; corner(reopen,17); setOn(reopen,true)
local body = new("Frame", {Size = UDim2.fromOffset(278,354), Position = UDim2.fromOffset(12,44), BackgroundTransparency = 1}, frame)
new("UIListLayout", {Padding = UDim.new(0,6), SortOrder = Enum.SortOrder.LayoutOrder}, body)
local right = new("Frame", {Size = UDim2.fromOffset(406,354), Position = UDim2.fromOffset(302,44), BackgroundTransparency = 1}, frame)
local tabs = new("Frame", {Size = UDim2.new(1,0,0,30), BackgroundTransparency = 1}, right)
new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0,6)}, tabs)
local cmdTab = button(tabs,"Ações",1,UDim2.new(1/5,-5,1,0))
local msgTab = button(tabs,"Aviso",2,UDim2.new(1/5,-5,1,0))
local chatTab = button(tabs,"Conversa",3,UDim2.new(1/5,-5,1,0))
local usersTab = button(tabs,"Servidores",4,UDim2.new(1/5,-5,1,0))
local tagsTab = button(tabs,"Tags",5,UDim2.new(1/5,-5,1,0))
local remTab = button(tabs,"Remotes",6,UDim2.new(0,0,1,0)); remTab.Visible = false
for _, tb in ipairs({cmdTab, msgTab, chatTab, usersTab, tagsTab}) do tb.TextSize = 12 end
local function pane()
 local p = new("Frame", {Size = UDim2.new(1,0,1,-38), Position = UDim2.fromOffset(0,38), BackgroundTransparency = 1},right)
 new("UIListLayout", {Padding = UDim.new(0,6), SortOrder = Enum.SortOrder.LayoutOrder},p)
 return p
end
local commandsPane=new("ScrollingFrame",{Size=UDim2.new(1,0,1,-38),Position=UDim2.fromOffset(0,38),BackgroundTransparency=1,BorderSizePixel=0,ScrollBarThickness=3,CanvasSize=UDim2.new(),AutomaticCanvasSize=Enum.AutomaticSize.Y},right)
new("UIListLayout",{Padding=UDim.new(0,6),SortOrder=Enum.SortOrder.LayoutOrder},commandsPane)
local messagePane=pane()
messagePane.Visible = false
local status = new("TextLabel", {Size = UDim2.new(1,-28,0,18), Position = UDim2.new(0,14,1,-24), BackgroundTransparency = 1, TextSize = 11, TextXAlignment = Enum.TextXAlignment.Left, TextColor3 = C.sub, TextTruncate = Enum.TextTruncate.AtEnd, Text = "pronto"}, frame)
local function setStatus(t,color) if not alive then return end; status.Text = t; status.TextColor3 = color or C.sub end
local minimized = false
local function setOpen(open)
 minimized = not open; frame.Visible = open; reopen.Visible = not open
 if open then uiScale.Scale = uiScale.Scale * 0.97; fitWindow() end
end
minBtn.MouseButton1Click:Connect(function() setOpen(false) end)
reopen.MouseButton1Click:Connect(function() setOpen(true) end)


local selected = {}        
local activeUsers = {}     
local loadingTargets = false
local openTagEditor
local targetRows={}
local targetEmpty=nil
local sessionStarts={}
local cachedActive={}
local requestedEditorId=nil
local tagReadSeq=0
local savingTag=false
local function sessionStart(value)
 local n=tonumber(value)
 if not n and type(value)=="string" then local ok,dt=pcall(DateTime.fromIsoDate,value);if ok and dt then n=dt.UnixTimestamp end end
 if n and n>100000000000 then n=n/1000 end
 return n and n>0 and n or nil
end
local function rememberSession(u)
 local started=sessionStart(u.sessionStartedAt or u.executionStartedAt or u.SessionStartedAt)
 if started then sessionStarts[tostring(u.userId)]={started=started,sessionId=u.sessionId} end
end
local function executionText(id)
 local rec=sessionStarts[tostring(id)]
 if not rec then return "tempo não informado pela API" end
 local seconds=math.floor(math.max(0,os.time()-rec.started))
 return string.format("Executando: %02d:%02d:%02d",math.floor(seconds/3600),math.floor(seconds/60)%60,seconds%60)
end
local function reconcileActive(cache,users,now)
 local seen={}
 for _,u in ipairs(users) do
  if type(u)=="table" and u.userId~=nil then
   local id=tostring(u.userId);seen[id]=true
   if u.online==false or u.executing==false then cache[id]=nil
   else
    local record=cache[id] or {user={},seen=now}
    for k,v in pairs(u) do record.user[k]=v end
    record.seen=now;record.missingAt=nil;cache[id]=record
   end
  end
 end
 local list={}
 for id,record in pairs(cache) do
  if not seen[id] then record.missingAt=record.missingAt or now end
  if record.missingAt and now-record.missingAt>=30 then cache[id]=nil
  else list[id]=record.user;table.insert(list,record.user) end
 end
 table.sort(list,function(x,y) return tostring(x.userId)<tostring(y.userId) end)
 return list
end
local function setChanged(obj,key,value) if obj[key]~=value then obj[key]=value end end

label(body, "PESSOAS", 1)
local targetTools = new("Frame", {Size = UDim2.new(1, 0, 0, 30), BackgroundTransparency = 1, LayoutOrder = 2}, body)
new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6)}, targetTools)
local nameBox = box(targetTools, "buscar nome ou @usuario...", 1, 30)
nameBox.Size = UDim2.new(1, -38, 1, 0)
local refreshPeople = button(targetTools, "↻", 2, UDim2.fromOffset(32, 30))
local targetInfo = label(body, "carregando...", 3, 16, C.sub)
targetInfo.TextTruncate = Enum.TextTruncate.AtEnd
local plist = new("ScrollingFrame", {Size = UDim2.new(1, 0, 0, 236), BackgroundColor3 = C.panel, BorderSizePixel = 0, ScrollBarThickness = 3, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y, LayoutOrder = 5}, body)
corner(plist, 8)
new("UIListLayout", {Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder}, plist)
new("UIPadding", {PaddingTop = UDim.new(0, 5), PaddingBottom = UDim.new(0, 5), PaddingLeft = UDim.new(0, 5), PaddingRight = UDim.new(0, 7)}, plist)
local selectionRow = new("Frame", {Size = UDim2.new(1, 0, 0, 28), BackgroundTransparency = 1, LayoutOrder = 6}, body)
new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6)}, selectionRow)
local clearBtn = button(selectionRow, "Limpar seleção", 1, UDim2.new(0.5, -3, 1, 0))
local gotoBtn = button(selectionRow, "📍 Ir até", 2, UDim2.new(0.5, -3, 1, 0))

local function userLabel(u)
	local display = tostring(u.displayName or "")
	local username = tostring(u.username or "")
	if display == "" then display = username end
	return display, username
end
local function findPlayers(query)
	query = tostring(query or ""):lower()
	local out = {}
	if query == "" then return out end
	for _, u in ipairs(activeUsers) do
		local display, username = userLabel(u)
		if username:lower():find(query, 1, true) or display:lower():find(query, 1, true) then table.insert(out, u) end
	end
	return out
end
local function selectedCount()
	local n = 0
	for id in pairs(selected) do
		if activeUsers[id] then n = n + 1 end
	end
	return n
end
local renderOnline
local refreshList
local function refreshTargets()
	local names = {}
	for id in pairs(selected) do
		local u = activeUsers[id]
		if u then
			local display = userLabel(u)
			table.insert(names, display)
		else
			selected[id] = nil
		end
	end
	table.sort(names)
	if #names == 0 then
		targetInfo.Text = tostring(#activeUsers) .. " online  ·  0 selecionados"
	else
		targetInfo.Text = tostring(#activeUsers) .. " online  ·  " .. tostring(#names) .. " selecionados"
	end
end
refreshList = function()
 local filter=nameBox.Text:lower()
 local shown=0
 local present={}
 for i,u in ipairs(activeUsers) do
  local id=tostring(u.userId);present[id]=true
  local display,username=userLabel(u)
  local match=filter=="" or username:lower():find(filter,1,true) or display:lower():find(filter,1,true) or ("server "..tostring(u.serverNumber or "")):find(filter,1,true)
  local rec=targetRows[id]
  if not rec then
   local row=new("TextButton",{Size=UDim2.new(1,0,0,76),BackgroundColor3=C.field,Text="",AutoButtonColor=false,LayoutOrder=i},plist)
   corner(row,7)
   local av=new("ImageLabel",{Size=UDim2.fromOffset(38,38),Position=UDim2.new(0,7,0.5,-19),BackgroundColor3=C.btn,Image=avatarUrl(u.userId)},row);corner(av,19)
   local function line(y,color,size) return new("TextLabel",{Size=UDim2.new(1,-86,0,16),Position=UDim2.fromOffset(53,y),BackgroundTransparency=1,Text="",TextColor3=color,TextSize=size,Font=Enum.Font.Gotham,TextXAlignment=Enum.TextXAlignment.Left,TextTruncate=Enum.TextTruncate.AtEnd},row) end
   rec={row=row,name=line(4,C.text,13),server=line(22,C.accent,11),game=line(39,C.sub,10),duration=line(56,C.sub,10)}
   rec.mark=new("TextLabel",{Size=UDim2.fromOffset(24,76),Position=UDim2.new(1,-29,0,0),BackgroundTransparency=1,Text="",TextColor3=C.accent,TextSize=18},row)
   targetRows[id]=rec
   row.MouseButton1Click:Connect(function() selected[id]=not selected[id] or nil;refreshTargets();refreshList() end)
  end
  setChanged(rec.row,"Visible",match and true or false)
  setChanged(rec.row,"LayoutOrder",i)
  setChanged(rec.row,"BackgroundColor3",selected[id] and C.selected or C.field)
  setChanged(rec.name,"Text",display.."  @"..username)
  setChanged(rec.server,"Text","Server "..tostring(u.serverNumber or "?").." • "..tostring(u.playerCount or 0).."/"..tostring(u.maxPlayers or 0))
  setChanged(rec.game,"Text",tostring(u.gameName or "Jogo desconhecido")..(u.sameServer and " • você está aqui" or ""))
  setChanged(rec.duration,"Text",executionText(id))
  setChanged(rec.mark,"Text",selected[id] and "✓" or "+")
  if match then shown+=1 end
 end
 for id,rec in pairs(targetRows) do if not present[id] then rec.row:Destroy();targetRows[id]=nil end end
 if not targetEmpty then targetEmpty=label(plist,"",9999,36,C.sub,Enum.Font.Gotham) end
 targetEmpty.Text=loadingTargets and "carregando..." or "nenhum usuario ativo encontrado"
 targetEmpty.Visible=shown==0
end
local function loadTargets()
	if loadingTargets or not connected then return end
	loadingTargets = true; if #activeUsers == 0 then refreshList() end
	task.spawn(function()
		local data, err = api("/admin/active", {issuerId = tostring(player.UserId), jobId = game.JobId, scope = "all"})
		loadingTargets = false
		if not alive then return end
		if not data then setStatus("erro ao carregar usuarios: " .. tostring(err), C.bad); refreshList(); return end
		if type(data.users)~="table" then setStatus("resposta incompleta; lista preservada",C.sub);return end
        activeUsers=reconcileActive(cachedActive,data.users,os.clock())
        for _,u in ipairs(activeUsers) do rememberSession(u) end
		refreshTargets(); refreshList()
        if renderOnline then renderOnline() end
		setStatus(tostring(data.total or #activeUsers) .. " usuario(s) usando o script", C.good)
	end)
end
local function getTargets()
	local list, needsGlobal = {}, false
	for id in pairs(selected) do
		local u = activeUsers[id]
		if u then
			table.insert(list, id)
			if not u.sameServer then needsGlobal = true end
		end
	end
	if #list == 0 then return nil, false end
	return list, needsGlobal
end
nameBox:GetPropertyChangedSignal("Text"):Connect(refreshList)
nameBox.FocusLost:Connect(function(enter)
	if not enter then return end
	local found = findPlayers(nameBox.Text)
	for _, u in ipairs(found) do selected[tostring(u.userId)] = true end
	setStatus(#found > 0 and ("selecionado(s): " .. #found) or "nenhum usuario ativo com esse nome", #found > 0 and C.good or C.bad)
	refreshTargets(); refreshList()
end)
refreshPeople.MouseButton1Click:Connect(loadTargets)
clearBtn.MouseButton1Click:Connect(function() selected = {}; refreshTargets(); refreshList() end)
gotoBtn.MouseButton1Click:Connect(function()
	local target
	for id in pairs(selected) do target = Players:GetPlayerByUserId(tonumber(id) or 0); if target then break end end
	local myChar = player.Character
	local root = target and target.Character and target.Character:FindFirstChild("HumanoidRootPart")
	if root and myChar then
		pcall(function() myChar:PivotTo(root.CFrame * CFrame.new(0, 0, 3)) end)
		setStatus("teleportado até " .. target.DisplayName, C.good)
	else
		setStatus("essa pessoa está em outro server", C.bad)
	end
end)


label(commandsPane, "PARÂMETROS", 10)
local argBox = box(commandsPane, "valor, segundos ou texto...", 11, 30)
label(commandsPane, "COMANDOS", 12)
local grid = new("Frame", {Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, LayoutOrder = 13}, commandsPane)
new("UIGridLayout", {CellSize = UDim2.new(1/3, -4, 0, 32), CellPadding = UDim2.fromOffset(6, 6), SortOrder = Enum.SortOrder.LayoutOrder}, grid)

local function splitArgs(def, raw)
	raw = tostring(raw or "")
	if raw == "" then return {} end
	if def.joinArgs then return {raw} end
	local out = {}
	for w in raw:gmatch("%S+") do table.insert(out, w) end
	return out
end

local function run(def, rawArgs, targetsOverride, scope)
	local targets, crossServer
	if targetsOverride then targets = targetsOverride else targets, crossServer = getTargets() end
	if not targets then setStatus("selecione pelo menos 1 usuario do script", C.bad); return end
	local finalScope = scope or (crossServer and "global" or "server")
	setStatus("enviando !" .. def.name .. "...")
	task.spawn(function()
		local data, err = sendCommand(def.name, splitArgs(def, rawArgs), targets, finalScope)
		if data then
			setStatus("!" .. def.name .. " → " .. tostring(data.targets) .. " jogador(es) com o script", C.good)
		else
			setStatus("erro: " .. tostring(err), C.bad)
		end
	end)
end

for i, def in ipairs(AdminCommands) do
	local compactNames = {kill="Matar",me="Trazer",freeze="Congelar",unfreeze="Descongelar",fling="Arremessar",flingfar="Arrem. longe",reload="Recarregar",trip="Derrubar",spin="Girar",jump="Pular",sit="Sentar",speed="Velocidade",blind="Cegar",shake="Tremer tela",dance="Dançar",say="Falar",rejoin="Reentrar",kick="Expulsar"}
	local b = button(grid, compactNames[def.name] or def.label, i)
	b.TextSize = 12
	b.MouseButton1Click:Connect(function() run(def, argBox.Text) end)
end


local function mountLocationCapture()
 local saved=""
 local dialog=new("Frame",{Size=UDim2.fromOffset(340,230),AnchorPoint=Vector2.new(0.5,0.5),Position=UDim2.fromScale(0.5,0.5),BackgroundColor3=C.panel,Visible=false,ZIndex=80},gui)
 corner(dialog,8)
 local title=new("TextLabel",{Size=UDim2.new(1,-24,0,26),Position=UDim2.fromOffset(12,10),BackgroundTransparency=1,Text="Localização atual · CFrame",TextColor3=C.text,Font=Enum.Font.GothamBold,TextSize=15,ZIndex=81},dialog)
 local value=new("TextBox",{Size=UDim2.new(1,-24,0,118),Position=UDim2.fromOffset(12,44),BackgroundColor3=C.field,TextColor3=C.text,Text="",TextWrapped=true,TextEditable=false,ClearTextOnFocus=false,MultiLine=true,TextSize=12,Font=Enum.Font.Code,TextXAlignment=Enum.TextXAlignment.Left,TextYAlignment=Enum.TextYAlignment.Top,ZIndex=81},dialog)
 local copy=button(dialog,"Copiar",1,UDim2.new(0.5,-18,0,32));copy.Position=UDim2.fromOffset(12,184);copy.ZIndex=82
 local close=button(dialog,"Fechar",2,UDim2.new(0.5,-18,0,32));close.Position=UDim2.new(0.5,6,0,184);close.ZIndex=82
 local last=box(commandsPane,"Última localização copiada",15,66,true);last.TextEditable=false
 local capture=button(commandsPane,"Salvar localização atual",14,UDim2.new(1,0,0,32))
 capture.Activated:Connect(function()
  local char=player.Character
  local root=char and char:FindFirstChild("HumanoidRootPart")
  if not root then setStatus("Personagem indisponível; tente depois de nascer.",C.bad);return end
  local parts={root.CFrame:GetComponents()}
  saved=LocationRules.cframe(parts)
  if not saved then setStatus("Não foi possível ler a localização.",C.bad);return end
  value.Text=saved;dialog.Visible=true
 end)
 copy.Activated:Connect(function()
  local clipboard=setclipboard or toclipboard
  if type(clipboard)~="function" then value:CaptureFocus();value.SelectionStart=1;value.CursorPosition=#value.Text+1;setStatus("Cópia automática indisponível; copie o texto selecionado.",C.bad);return end
  local ok,result=pcall(clipboard,saved)
  if not ok or result==false then setStatus("Não foi possível copiar; a janela continua aberta.",C.bad);return end
  last.Text=saved;dialog.Visible=false;setStatus("Localização copiada.",C.good)
 end)
 close.Activated:Connect(function() dialog.Visible=false end)
end
mountLocationCapture()
label(messagePane, "MENSAGEM", 20)
local msgBox = box(messagePane, "escreva a mensagem...", 21, 140, true)
local scopeRow = new("Frame", {Size = UDim2.new(1, 0, 0, 28), BackgroundTransparency = 1, LayoutOrder = 22}, messagePane)
new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6)}, scopeRow)
local bScopeServer = button(scopeRow, "Só este server", 1, UDim2.new(0.5, -3, 1, 0))
local bScopeAll = button(scopeRow, "Todos os servers", 2, UDim2.new(0.5, -3, 1, 0))
local msgScope = "server"
local function refreshScope() setOn(bScopeServer, msgScope == "server"); setOn(bScopeAll, msgScope == "global") end
bScopeServer.MouseButton1Click:Connect(function() msgScope = "server"; refreshScope() end)
bScopeAll.MouseButton1Click:Connect(function() msgScope = "global"; refreshScope() end)
local sendMsg = button(messagePane, "Enviar mensagem", 23, UDim2.new(1,0,0,32))
label(messagePane, "SCRIPT", 24)
local reloadAll = button(messagePane, "♻️ Recarregar o script de TODO MUNDO", 25, UDim2.new(1,0,0,32))
local reloadArm = 0
reloadAll.MouseButton1Click:Connect(function()
	if os.clock() - reloadArm > 3 then reloadArm = os.clock(); reloadAll.Text = "Toque de novo para confirmar"; setStatus("toque de novo para recarregar em todo mundo", C.accent); return end
	reloadArm = 0; reloadAll.Text = "♻️ Recarregar o script de TODO MUNDO"
	setStatus("recarregando script de todos...")
	task.spawn(function()
		local data, err = sendCommand("reload", {}, "all", "global")
		if data then setStatus("reload enviado para " .. tostring(data.targets) .. " pessoa(s)", C.good) else setStatus("erro: " .. tostring(err), C.bad) end
	end)
end)
setOn(sendMsg,true)
sendMsg.MouseButton1Click:Connect(function()
	local text = msgBox.Text:gsub("^%s+", ""):gsub("%s+$", "")
	if text == "" then setStatus("escreva uma mensagem", C.bad); return end
	if #text > 200 then text = text:sub(1, 200) end
	setStatus("enviando mensagem...")
	task.spawn(function()
		local data, err = sendCommand("announce", {text, "8"}, "all", msgScope)
		if data then
			setStatus("mensagem enviada para " .. tostring(data.targets) .. " pessoa(s)", C.good)
			msgBox.Text = ""
		else
			setStatus("erro: " .. tostring(err), C.bad)
		end
	end)
end)


local onlineBtn = usersTab

local online = new("Frame", {Size = UDim2.new(1,0,1,-38), Position = UDim2.fromOffset(0,38), BackgroundColor3 = C.bg, Visible = false}, right)
corner(online, 8)
local oTitle = new("TextLabel", {Size = UDim2.new(1, -50, 0, 22), Position = UDim2.fromOffset(12, 8), BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 14, TextColor3 = C.accent, TextXAlignment = Enum.TextXAlignment.Left, Text = "Executando agora"}, online)
local oCount = new("TextLabel", {Size = UDim2.new(1, -24, 0, 16), Position = UDim2.fromOffset(12, 30), BackgroundTransparency = 1, Font = Enum.Font.Gotham, TextSize = 12, TextColor3 = C.text, TextXAlignment = Enum.TextXAlignment.Left, Text = "carregando..."}, online)
local oClose = button(online, "✕", 0, UDim2.fromOffset(26, 24)); oClose.Position = UDim2.new(1, -34, 0, 8)
local oRow = new("Frame", {Size = UDim2.new(1, -24, 0, 26), Position = UDim2.fromOffset(12, 52), BackgroundTransparency = 1}, online)
new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6)}, oRow)
local oServer = button(oRow, "Este server", 1, UDim2.new(0.36, -4, 1, 0))
local oAll = button(oRow, "Todos", 2, UDim2.new(0.28, -4, 1, 0))
local oRefresh = button(oRow, "↻ Atualizar", 3, UDim2.new(0.36, -4, 1, 0))
local oList = new("ScrollingFrame", {Size = UDim2.new(1, -24, 1, -92), Position = UDim2.fromOffset(12, 84), BackgroundColor3 = C.panel, BorderSizePixel = 0, ScrollBarThickness = 3, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y}, online)
corner(oList, 8)
new("UIListLayout", {Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder}, oList)
new("UIPadding", {PaddingTop = UDim.new(0, 4), PaddingBottom = UDim.new(0, 4), PaddingLeft = UDim.new(0, 4), PaddingRight = UDim.new(0, 6)}, oList)

local onlineScope = "server"
local loadingOnline = false
local onlineRows={}
local onlineRequestSeq=0
local loadOnline
renderOnline=function()
 local present={};local count=0;local same=0
 for _,u in ipairs(activeUsers) do
  local id=tostring(u.userId)
  if onlineScope=="all" or u.sameServer then
   present[id]=true;count+=1;if u.sameServer then same+=1 end
   local rec=onlineRows[id]
   if not rec then
    local row=new("Frame",{Size=UDim2.new(1,0,0,66),BackgroundColor3=C.field},oList);corner(row)
    local av=new("ImageLabel",{Size=UDim2.fromOffset(34,34),Position=UDim2.fromOffset(5,6),BackgroundColor3=C.btn,Image=avatarUrl(id)},row);corner(av,17)
    local function text(y) return new("TextLabel",{Position=UDim2.fromOffset(45,y),Size=UDim2.new(1,-150,0,14),BackgroundTransparency=1,Text="",TextColor3=C.text,TextSize=11,TextXAlignment=Enum.TextXAlignment.Left,TextTruncate=Enum.TextTruncate.AtEnd},row) end
    rec={row=row,name=text(4),tag=text(19),server=text(34),duration=text(49)}
    local tag=button(row,"TAG",0,UDim2.fromOffset(42,24));tag.Position=UDim2.new(1,-96,0.5,-12)
    tag.MouseButton1Click:Connect(function() if openTagEditor then openTagEditor(id) end end)
    local tp=button(row,"TP",0,UDim2.fromOffset(42,24));tp.Position=UDim2.new(1,-48,0.5,-12);rec.tp=tp
    tp.MouseButton1Click:Connect(function()
     local current=activeUsers[id]
     if not current or current.sameServer or not tonumber(current.placeId) or not current.jobId then return end
     task.spawn(function()
      local ok=pcall(TeleportService.TeleportToPlaceInstance,TeleportService,tonumber(current.placeId),tostring(current.jobId),player)
      if not ok then setStatus("não foi possível entrar no servidor",C.bad) end
     end)
    end)
    onlineRows[id]=rec
   end
   local display,username=userLabel(u)
   setChanged(rec.row,"LayoutOrder",count)
   setChanged(rec.name,"Text",display.." @"..username)
   setChanged(rec.tag,"Text","Tag: "..tostring(u.Tag or "V User"))
   setChanged(rec.server,"Text","Server "..tostring(u.serverNumber or "?").." • "..tostring(u.gameName or ""))
   setChanged(rec.duration,"Text",executionText(id))
   setChanged(rec.tp,"Visible",not u.sameServer and tonumber(u.placeId)~=nil and u.jobId~=nil and u.jobId~="")
  end
 end
 for id,rec in pairs(onlineRows) do if not present[id] then rec.row:Destroy();onlineRows[id]=nil end end
 oCount.Text=string.format("Exibidos: %d | Neste server: %d",count,same)
end
loadOnline=function()
 setOn(oServer,onlineScope=="server");setOn(oAll,onlineScope=="all")
 renderOnline()
 loadTargets()
end


local ADMIN_NAME = "Valença"
local chatPane = pane(); chatPane.Visible = false
local chatState = {userId = nil, since = 0, open = false, seen = {}}
local chatHead = label(chatPane, "CONVERSA — selecione 1 pessoa na lista", 30, 18, C.accent)
chatHead.TextTruncate = Enum.TextTruncate.AtEnd
local chatList = new("ScrollingFrame", {Size = UDim2.new(1, 0, 0, 172), BackgroundColor3 = C.panel, BorderSizePixel = 0, ScrollBarThickness = 3, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y, LayoutOrder = 31}, chatPane)
corner(chatList, 8)
new("UIListLayout", {Padding = UDim.new(0, 5), SortOrder = Enum.SortOrder.LayoutOrder}, chatList)
new("UIPadding", {PaddingTop = UDim.new(0, 6), PaddingBottom = UDim.new(0, 6), PaddingLeft = UDim.new(0, 6), PaddingRight = UDim.new(0, 8)}, chatList)
local chatBox = box(chatPane, "escreva para a pessoa...", 32, 30)
local chatRow = new("Frame", {Size = UDim2.new(1, 0, 0, 30), BackgroundTransparency = 1, LayoutOrder = 33}, chatPane)
new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6)}, chatRow)
local chatSend = button(chatRow, "Enviar", 1, UDim2.new(0.6, -3, 1, 0)); setOn(chatSend, true)
local chatEnd = button(chatRow, "✕ Encerrar", 2, UDim2.new(0.4, -3, 1, 0))
local customSound = false
local soundBtn = button(chatPane, "☐ Som customizado", 34, UDim2.new(1, 0, 0, 24))
soundBtn.TextSize = 12
soundBtn.MouseButton1Click:Connect(function()
	customSound = not customSound
	soundBtn.Text = (customSound and "☑" or "☐") .. " Som customizado"
	setOn(soundBtn, customSound)
end)

local phoneGui = new("Frame", {AnchorPoint = Vector2.new(1, 1), Position = UDim2.new(1, -16, 1, -16), Size = UDim2.fromOffset(250, 360), BackgroundColor3 = C.bg, Visible = false, Active = true, Draggable = true}, gui)
corner(phoneGui, 22); stroke(phoneGui, C.accent)
new("Frame", {AnchorPoint = Vector2.new(0.5, 0), Position = UDim2.new(0.5, 0, 0, 6), Size = UDim2.fromOffset(60, 5), BackgroundColor3 = C.btn}, phoneGui)
local phoneDot = new("Frame", {Position = UDim2.fromOffset(14, 30), Size = UDim2.fromOffset(8, 8), BackgroundColor3 = C.good}, phoneGui); corner(phoneDot, 8)
local phoneName = new("TextLabel", {Position = UDim2.fromOffset(28, 18), Size = UDim2.new(1, -70, 0, 20), BackgroundTransparency = 1, Text = "", TextColor3 = C.accent, Font = Enum.Font.GothamBold, TextSize = 15, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd}, phoneGui)
local phoneDuration=new("TextLabel", {Position = UDim2.fromOffset(28, 37), Size = UDim2.new(1, -40, 0, 14), BackgroundTransparency = 1, Text = "conversa privada (ADM)", TextColor3 = C.sub, Font = Enum.Font.Gotham, TextSize = 10, TextXAlignment = Enum.TextXAlignment.Left,TextTruncate=Enum.TextTruncate.AtEnd}, phoneGui)
local phoneX = button(phoneGui, "×", 0, UDim2.fromOffset(28, 28)); phoneX.Position = UDim2.new(1, -40, 0, 18); corner(phoneX, 14); phoneX.TextSize = 18
local phoneList = new("ScrollingFrame", {Position = UDim2.fromOffset(10, 62), Size = UDim2.new(1, -20, 1, -114), BackgroundColor3 = C.panel, BorderSizePixel = 0, ScrollBarThickness = 2, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y}, phoneGui)
corner(phoneList, 12)
new("UIListLayout", {Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder}, phoneList)
new("UIPadding", {PaddingTop = UDim.new(0, 8), PaddingBottom = UDim.new(0, 8), PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8)}, phoneList)
local phoneBox = box(phoneGui, "mensagem...", 0, 34); phoneBox.Position = UDim2.new(0, 10, 1, -44); phoneBox.Size = UDim2.new(1, -66, 0, 34); corner(phoneBox, 17)
local phoneSend = button(phoneGui, ">", 0, UDim2.fromOffset(40, 34)); phoneSend.Position = UDim2.new(1, -50, 1, -44); corner(phoneSend, 17); setOn(phoneSend, true); phoneSend.TextSize = 18
local phoneSeen = {}
local phoneFor = nil
phoneX.MouseButton1Click:Connect(function() phoneGui.Visible = false end)

local function chatClear()
	for _, c in ipairs(chatList:GetChildren()) do if c:IsA("Frame") then c:Destroy() end end
	chatState.seen = {}; chatState.since = 0
end
local function phoneReset()
	for _, c in ipairs(phoneList:GetChildren()) do if c:IsA("Frame") then c:Destroy() end end
	phoneSeen = {}
end
local function chatBubble(m)
	if chatState.seen[m.id] then return end
	chatState.seen[m.id] = true
	local mine = m.from == "admin"
	local row = new("Frame", {Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, LayoutOrder = tonumber(m.id) or 0}, chatList)
	local b = new("TextLabel", {AnchorPoint = Vector2.new(mine and 1 or 0, 0), Position = UDim2.new(mine and 1 or 0, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.XY, Size = UDim2.new(), BackgroundColor3 = mine and C.accent or C.field, TextColor3 = mine and C.ink or C.text, Text = tostring(m.text or ""), TextWrapped = true, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left, Font = Enum.Font.Gotham}, row)
	new("UISizeConstraint", {MaxSize = Vector2.new(280, 1000)}, b)
	new("UIPadding", {PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, 5), PaddingBottom = UDim.new(0, 5)}, b)
	corner(b, 9)
	task.defer(function() chatList.CanvasPosition = Vector2.new(0, 1e6) end)
	if not phoneSeen[m.id] then
		phoneSeen[m.id] = true
		local prow = new("Frame", {Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, LayoutOrder = tonumber(m.id) or 0}, phoneList)
		local pb = new("TextLabel", {AnchorPoint = Vector2.new(mine and 1 or 0, 0), Position = UDim2.new(mine and 1 or 0, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.XY, Size = UDim2.new(), BackgroundColor3 = mine and C.accent or C.field, TextColor3 = mine and C.ink or C.text, Text = tostring(m.text or ""), TextWrapped = true, TextSize = 13, TextXAlignment = Enum.TextXAlignment.Left, Font = Enum.Font.GothamMedium}, prow)
		new("UISizeConstraint", {MaxSize = Vector2.new(180, 1000)}, pb)
		new("UIPadding", {PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 8), PaddingTop = UDim.new(0, 5), PaddingBottom = UDim.new(0, 5)}, pb)
		corner(pb, 10)
		task.defer(function() phoneList.CanvasPosition = Vector2.new(0, 1e6) end)
	end
end
local function chatTarget()
	local ids = {}
	for id in pairs(selected) do table.insert(ids, id) end
	if #ids ~= 1 then return nil end
	return ids[1]
end
local function chatNameOf(id)
	for _, u in ipairs(activeUsers) do if tostring(u.userId) == id then local d, n = userLabel(u); return d .. " (@" .. n .. ")" end end
	return id
end
local chatBusy = false
local chatSyncInner
local function chatSync()
	if chatBusy then return end
	chatBusy = true
	local ok, err = pcall(chatSyncInner)
	chatBusy = false
end
chatSyncInner = function()
	local id = chatTarget()
	if id ~= chatState.userId then
		chatState.userId = id; chatState.open = false; chatClear()
		if id and phoneGui.Visible and phoneFor ~= id then phoneFor = id; phoneReset(); phoneName.Text = chatNameOf(id) end
	end
	if not id then chatHead.Text = "CONVERSA — selecione só 1 pessoa na lista"; return end
	local data = api("/admin/chat/poll", {issuerId = tostring(player.UserId), userId = id, since = chatState.since})
	if not data or chatState.userId ~= id then return end
	chatState.open = data.open == true
	chatHead.Text = (chatState.open and "● " or "○ ") .. chatNameOf(id) .. (chatState.open and "" or "  (sem conversa aberta)")
	chatHead.TextColor3 = chatState.open and C.good or C.accent
    if type(data.user)=="table" then rememberSession(data.user) end
    phoneDuration.Text=executionText(id)
	for _, m in ipairs(data.messages or {}) do
		if (tonumber(m.id) or 0) > chatState.since then chatState.since = tonumber(m.id) end
		chatBubble(m)
	end
end
local function chatSendText(raw, forcedId)
	local id = forcedId or chatTarget()
	if not id then setStatus("selecione só 1 pessoa para conversar", C.bad); return end
	local text = tostring(raw or ""):gsub("^%s+", ""):gsub("%s+$", "")
	if text == "" then return end
	if phoneFor ~= id then phoneFor = id; phoneReset() end
	phoneName.Text = chatNameOf(id)
	phoneGui.Visible = true
	task.spawn(function()
		local data, err = api("/admin/chat/send", {issuerId = tostring(player.UserId), userId = id, text = text:sub(1, 200), adminName = ADMIN_NAME, customSound = customSound})
		if data then
			if chatState.userId ~= id then chatState.userId = id; chatClear() end
			chatState.open = true
			if data.message then chatBubble(data.message) end
			setStatus("mensagem enviada", C.good)
			chatSync()
		else setStatus("erro: " .. tostring(err), C.bad) end
	end)
end
local function chatDoSend() local t = chatBox.Text; chatBox.Text = ""; chatSendText(t) end
local function phoneDoSend() local t = phoneBox.Text; phoneBox.Text = ""; chatSendText(t, phoneFor) end
phoneSend.MouseButton1Click:Connect(phoneDoSend)
phoneBox.FocusLost:Connect(function(enter) if enter then phoneDoSend() end end)
chatSend.MouseButton1Click:Connect(chatDoSend)
chatBox.FocusLost:Connect(function(enter) if enter then chatDoSend() end end)
chatEnd.MouseButton1Click:Connect(function()
	local id = chatTarget(); if not id then return end
	task.spawn(function()
		api("/admin/chat/close", {issuerId = tostring(player.UserId), userId = id})
		chatState.open = false; setStatus("conversa encerrada", C.sub); chatSync()
	end)
end)
task.spawn(function()
	while alive do
		task.wait(1.5)
		if alive and ((frame.Visible and chatPane.Visible) or phoneGui.Visible) then pcall(chatSync) end
	end
end)


local tagsPane, tagResults, loadTagList, loadTagUser, picker
local function buildTagEditor()
local TAG_EFFECTS = {
	{"default", "Padrao", "brilho suave (quem nao escolheu efeito)", ""},
	{"glitch", "Glitch", "fatias RGB, letras corrompendo, scanline", ""},
	{"angel", "Anjo", "asas batendo + aureola", ""},
	{"demon", "Demonio", "asas de morcego + chifres", ""},
	{"devil", "Devil", "chifres escuros, sombra tremendo", ""},
	{"crown", "Coroa", "coroa dourada com joias", ""},
	{"void", "Void", "letras se dissolvem, tentaculos", ""},
	{"blackhole", "Buraco negro", "puxa as letras e cospe de volta", ""},
	{"scramble", "Embaralhado", "letras trocam de lugar", ""},
	{"mystery", "Misteriosa", "???? com o nome sendo revelado", ""},
	{"rainbow", "Arco-iris", "onda RGB letra por letra", ""},
	{"ocean", "Oceano", "letras em onda + bolhas", ""},
	{"matrix", "Matrix", "chuva de codigo + decode", ""},
	{"neon", "Neon", "letreiro com moldura piscando", ""},
	{"cyber", "Cyber", "HUD com colchetes e scan", ""},
	{"vampire", "Vampiro", "sangue escorrendo + morcegos", ""},
	{"toxic", "Toxico", "gosma escorrendo + bolhas", ""},
	{"love", "Amor", "coracoes batendo e subindo", ""},
	{"sakura", "Sakura", "petalas caindo", ""},
	{"phoenix", "Fenix", "asas de fogo + chamas", ""},
	{"celestial", "Celestial", "constelacao + estrelas orbitando", "aura"},
	{"inferno", "Inferno", "chamas atras do nome", "aura"},
	{"frost", "Frost", "cristais de gelo + brilho", "aura"},
	{"phantom", "Phantom", "fantasma flutuando + orbes", "aura"},
	{"thunder", "Thunder", "raios cortando a tag", "aura"},
	{"absorbed", "ABSORBED", "letras sugadas por um orbe", "ex+aura"},
	{"singularity", "SINGULARITY", "colapso total + onda de choque", "ex+aura"},
	{"corrupted", "C0RRUPTED", "glitch no dobro da forca", "ex"},
	{"seraph", "SERAPH", "4 asas + aureola dupla", "ex"},
	{"overlord", "OVERLORD", "asas + 3 pares de chifres", "ex"},
	{"eclipse", "ECLIPSE", "lua cobrindo o sol atras do nome", "ex"},
}
local TAG_FONTS = {"GothamBold", "GothamBlack", "Gotham", "GothamMedium", "Code", "Arcade", "Bangers", "Cartoon", "Fantasy", "Highway", "LuckiestGuy", "Michroma", "Oswald", "PermanentMarker", "Sarpanch", "SciFi", "Creepster", "Antique", "Garamond", "FredokaOne", "Bodoni", "Nunito", "Ubuntu", "GrenzeGotisch", "SpecialElite", "DenkOne", "TitilliumWeb", "RobotoMono", "AmaticSC", "Kalam"}
local PALETTE = {
	{"Branco", "255,255,255"}, {"Preto", "15,15,15"}, {"Cinza", "150,150,155"}, {"Prata", "200,205,215"},
	{"Vermelho", "255,40,40"}, {"Vinho", "110,10,30"}, {"Laranja", "255,130,20"}, {"Amarelo", "255,225,40"},
	{"Dourado", "255,200,60"}, {"Verde", "60,220,90"}, {"Verde limao", "160,255,40"}, {"Verde escuro", "20,120,50"},
	{"Ciano", "40,235,255"}, {"Azul claro", "120,190,255"}, {"Azul", "40,110,255"}, {"Azul escuro", "20,40,140"},
	{"Roxo", "150,70,255"}, {"Lilas", "200,160,255"}, {"Rosa", "255,110,190"}, {"Rosa claro", "255,190,220"},
	{"Magenta", "255,40,200"}, {"Marrom", "130,80,40"}, {"Bege", "235,210,170"}, {"Turquesa", "40,210,180"},
}
local effectInfo = {}
for _, e in ipairs(TAG_EFFECTS) do effectInfo[e[1]] = e end
local function parseColor(s)
	s = tostring(s or ""):gsub("%s", "")
	local hex = s:match("^#?(%x%x%x%x%x%x)$")
	if hex then return Color3.fromRGB(tonumber(hex:sub(1, 2), 16), tonumber(hex:sub(3, 4), 16), tonumber(hex:sub(5, 6), 16)) end
	local r, g, b = s:match("^(%d+)[,;](%d+)[,;](%d+)$")
	if r then return Color3.fromRGB(math.clamp(tonumber(r), 0, 255), math.clamp(tonumber(g), 0, 255), math.clamp(tonumber(b), 0, 255)) end
	return nil
end

tagsPane = new("ScrollingFrame", {Size = UDim2.new(1, 0, 1, -38), Position = UDim2.fromOffset(0, 38), BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 3, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y, Visible = false}, right)
new("UIListLayout", {Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder}, tagsPane)
new("UIPadding", {PaddingRight = UDim.new(0, 6), PaddingBottom = UDim.new(0, 6)}, tagsPane)
local function hrow(order, height)
	local f = new("Frame", {Size = UDim2.new(1, 0, 0, height or 28), BackgroundTransparency = 1, LayoutOrder = order}, tagsPane)
	new("UIListLayout", {FillDirection = Enum.FillDirection.Horizontal, Padding = UDim.new(0, 6), SortOrder = Enum.SortOrder.LayoutOrder}, f)
	return f
end
local searchRow = hrow(1)
local tagSearch = box(searchRow, "buscar no banco: nome, @user, id ou tag...", 1, 28)
tagSearch.Size = UDim2.new(1, -112, 1, 0)
local tagRefresh = button(searchRow, "↻", 2, UDim2.fromOffset(32, 28))
local tagUseSel = button(searchRow, "Selecionado", 3, UDim2.fromOffset(68, 28)); tagUseSel.TextSize = 11
tagResults = new("ScrollingFrame", {Size = UDim2.new(1, 0, 0, 118), BackgroundColor3 = C.panel, BorderSizePixel = 0, ScrollBarThickness = 3, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y, LayoutOrder = 2}, tagsPane)
corner(tagResults, 8)
new("UIListLayout", {Padding = UDim.new(0, 4), SortOrder = Enum.SortOrder.LayoutOrder}, tagResults)
new("UIPadding", {PaddingTop = UDim.new(0, 4), PaddingBottom = UDim.new(0, 4), PaddingLeft = UDim.new(0, 4), PaddingRight = UDim.new(0, 6)}, tagResults)
local edHead = label(tagsPane, "EDITOR — escolha alguem na lista", 3, 18, C.accent)
edHead.TextTruncate = Enum.TextTruncate.AtEnd
local tagName = box(tagsPane, "texto da tag (ate 24 letras)", 4, 28)
local fxRow = hrow(5)
local effectBtn = button(fxRow, "Efeito: default", 1, UDim2.new(0.62, -3, 1, 0))
local fontBtn = button(fxRow, "Fonte: GothamBold", 2, UDim2.new(0.38, -3, 1, 0)); fontBtn.TextSize = 12
local colRow = hrow(6)
local cor1Box = box(colRow, "cor 1 (255,255,255)", 1, 28); cor1Box.Size = UDim2.new(0.5, -3, 1, 0)
local cor2Box = box(colRow, "cor 2 (145,225,255)", 2, 28); cor2Box.Size = UDim2.new(0.5, -3, 1, 0)
local function swatch(b)
	local s = new("Frame", {AnchorPoint = Vector2.new(1, 0.5), Position = UDim2.new(1, 2, 0.5, 0), Size = UDim2.fromOffset(16, 16), BackgroundColor3 = Color3.new(1, 1, 1)}, b)
	corner(s, 4); stroke(s)
	local function upd() local c = parseColor(b.Text); s.BackgroundColor3 = c or C.btn; s.BackgroundTransparency = c and 0 or 0.6 end
	b:GetPropertyChangedSignal("Text"):Connect(upd); upd()
end
swatch(cor1Box); swatch(cor2Box)
local tgtRow = hrow(7, 22)
local bCor1 = button(tgtRow, "Paleta → Cor 1", 1, UDim2.new(0.5, -3, 1, 0)); bCor1.TextSize = 11
local bCor2 = button(tgtRow, "Paleta → Cor 2", 2, UDim2.new(0.5, -3, 1, 0)); bCor2.TextSize = 11
local palTarget = 1
local function refreshPalTarget() setOn(bCor1, palTarget == 1); setOn(bCor2, palTarget == 2) end
bCor1.MouseButton1Click:Connect(function() palTarget = 1; refreshPalTarget() end)
bCor2.MouseButton1Click:Connect(function() palTarget = 2; refreshPalTarget() end)
local palGrid = new("Frame", {Size = UDim2.new(1, 0, 0, 0), AutomaticSize = Enum.AutomaticSize.Y, BackgroundTransparency = 1, LayoutOrder = 8}, tagsPane)
new("UIGridLayout", {CellSize = UDim2.fromOffset(28, 22), CellPadding = UDim2.fromOffset(4, 4), SortOrder = Enum.SortOrder.LayoutOrder}, palGrid)
for i, p in ipairs(PALETTE) do
	local sw = new("TextButton", {BackgroundColor3 = parseColor(p[2]), Text = "", AutoButtonColor = true, LayoutOrder = i}, palGrid)
	corner(sw, 5); stroke(sw)
	sw.MouseButton1Click:Connect(function()
		if palTarget == 1 then cor1Box.Text = p[2] else cor2Box.Text = p[2] end
		setStatus("cor " .. palTarget .. ": " .. p[1], C.sub)
	end)
end
local flagRow = hrow(9)
local plusBtn = button(flagRow, "Plus: não", 1, UDim2.new(1 / 4, -4, 1, 0))
local coinBtn = button(flagRow, "Coin: não", 2, UDim2.new(1 / 4, -4, 1, 0))
local activeBtn = button(flagRow, "Tag ativa", 3, UDim2.new(1 / 4, -4, 1, 0))
local banBtn = button(flagRow, "Bloqueado: não", 4, UDim2.new(1 / 4, -4, 1, 0))
coinBtn.TextSize = 12; plusBtn.TextSize = 12; activeBtn.TextSize = 12; banBtn.TextSize = 12
local actRow = hrow(10, 32)
local saveTag = button(actRow, "Salvar no banco", 1, UDim2.new(0.5, -4, 1, 0)); setOn(saveTag, true)
local resetTag = button(actRow, "Resetar tag", 2, UDim2.new(0.25, -4, 1, 0)); resetTag.TextSize = 12
local reloadTag = button(actRow, "Recarregar", 3, UDim2.new(0.25, -4, 1, 0)); reloadTag.TextSize = 12
local edNote = label(tagsPane, "", 11, 30, C.sub, Enum.Font.Gotham)
edNote.TextWrapped = true; edNote.TextSize = 11

local edit = {userId = nil, effect = "default", font = "GothamBold", Plus = false, CoinCliker = false, Active = true, blacklisted = false}
local function refreshEditor()
	local info = effectInfo[edit.effect]
	effectBtn.Text = "Efeito: " .. (info and info[2] or edit.effect) .. ((info and info[4] ~= "") and ("  [" .. info[4] .. "]") or "")
	fontBtn.Text = "Fonte: " .. edit.font
	plusBtn.Text = edit.Plus and "Plus: SIM" or "Plus: não"
	plusBtn:SetAttribute("on", edit.Plus); plusBtn.BackgroundColor3 = edit.Plus and C.plus or C.btn
	coinBtn.Text = edit.CoinCliker and "Coin: SIM" or "Coin: não"
	coinBtn:SetAttribute("on", edit.CoinCliker); coinBtn.BackgroundColor3 = edit.CoinCliker and C.plus or C.btn
	activeBtn.Text = edit.Active and "Tag ativa" or "Tag desligada"; setOn(activeBtn, edit.Active)
	banBtn.Text = edit.blacklisted and "Bloqueado: SIM" or "Bloqueado: não"
	banBtn:SetAttribute("on", edit.blacklisted); banBtn.BackgroundColor3 = edit.blacklisted and C.bad or C.btn
	local ex = info and info[4]:find("ex")
	edNote.Text = ex and "Exclusiva: o script trava o texto e as cores dessa tag (o que voce digitar aqui fica salvo mas nao aparece)." or (info and info[4] == "aura" and "Essa tag tem mini aura no personagem." or "Deixe as cores vazias pra usar as cores padrao do efeito.")
end
local function fillEditor(u)
	edit.userId = tostring(u.userId)
    requestedEditorId=edit.userId
    rememberSession(u)
	edit.effect = tostring(u.TagEffect or "default"):lower()
	edit.font = tostring(u.TagFont or "GothamBold")
	edit.Plus = u.Plus == true; edit.CoinCliker = u.CoinCliker == true; edit.Active = u.Active ~= false; edit.blacklisted = u.blacklisted == true
	local name = tostring(u.displayName ~= "" and u.displayName or u.username or "")
	edHead.Text = "EDITANDO: " .. (name ~= "" and name or "?") .. (u.username and u.username ~= "" and ("  @" .. u.username) or "") .. "  ·  " .. edit.userId .. (u.online and "  ·  online" or "")
	tagName.Text = tostring(u.Tag or "")
	cor1Box.Text = tostring(u.Cor1Tag or ""); cor2Box.Text = tostring(u.Cor2Tag or "")
	refreshEditor()
end
local loadingTagList = false
loadTagList = function()
	if loadingTagList then return end
	loadingTagList = true
	task.spawn(function()
		local data, err = api("/admin/players", {issuerId = tostring(player.UserId), query = tagSearch.Text, limit = 120})
		loadingTagList = false
		if not alive then return end
		for _, c in ipairs(tagResults:GetChildren()) do if c:IsA("GuiObject") then c:Destroy() end end
		if not data then label(tagResults, "erro: " .. tostring(err), 1, 30, C.bad, Enum.Font.Gotham); return end
		if #(data.users or {}) == 0 then label(tagResults, "ninguem encontrado", 1, 30, C.sub, Enum.Font.Gotham) end
		for i, u in ipairs(data.users or {}) do
			local on = edit.userId == tostring(u.userId)
			local row = new("TextButton", {Size = UDim2.new(1, 0, 0, 36), BackgroundColor3 = on and C.selected or C.field, Text = "", AutoButtonColor = false, LayoutOrder = i}, tagResults)
			corner(row, 6)
			local av = new("ImageLabel", {Size = UDim2.fromOffset(26, 26), Position = UDim2.new(0, 5, 0.5, -13), BackgroundColor3 = C.btn, Image = avatarUrl(u.userId)}, row)
			corner(av, 13)
			local dotc = new("Frame", {Size = UDim2.fromOffset(6, 6), Position = UDim2.fromOffset(25, 24), BackgroundColor3 = u.online and C.good or C.sub}, row); corner(dotc, 3)
			local name = tostring(u.displayName ~= "" and u.displayName or u.username or u.userId)
			new("TextLabel", {Size = UDim2.new(1, -90, 0, 16), Position = UDim2.fromOffset(38, 3), BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 12, TextColor3 = C.text, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd, Text = name .. (u.username ~= "" and ("  @" .. u.username) or "")}, row)
			new("TextLabel", {Size = UDim2.new(1, -90, 0, 14), Position = UDim2.fromOffset(38, 19), BackgroundTransparency = 1, Font = Enum.Font.Gotham, TextSize = 11, TextColor3 = parseColor(u.Cor1Tag) or C.sub, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd, Text = tostring(u.Tag or "-") .. "  ·  " .. tostring(u.TagEffect or "default")}, row)
			if u.Plus then new("TextLabel", {Size = UDim2.fromOffset(40, 16), Position = UDim2.new(1, -46, 0.5, -8), BackgroundColor3 = C.plus, Font = Enum.Font.GothamBold, TextSize = 10, TextColor3 = C.text, Text = "PLUS"}, row) end
			row.MouseButton1Click:Connect(function() tagReadSeq+=1;fillEditor(u);loadTagList() end)
		end
		setStatus(tostring(data.total or 0) .. " perfil(s) no banco" .. (tagSearch.Text ~= "" and " com esse filtro" or ""), C.sub)
	end)
end
loadTagUser = function(id)
 id=tostring(id or "")
 if id=="" then return end
 requestedEditorId=id
 tagReadSeq+=1
 local seq=tagReadSeq
 edit.userId=nil
 edHead.Text="Carregando perfil..."
	task.spawn(function()
		local data, err = api("/admin/player/get", {issuerId = tostring(player.UserId), userId = id})
		if not alive then return end
		if seq~=tagReadSeq or requestedEditorId~=id then return end
        if data and type(data.user)=="table" and tostring(data.user.userId)==id then fillEditor(data.user);setStatus("perfil carregado",C.good) else setStatus("erro: "..tostring(err),C.bad) end
	end)
end
local function saveTagPatch(patch,okMsg)
 if not edit.userId then setStatus("escolha alguem primeiro",C.bad);return end
 if savingTag then setStatus("aguarde o salvamento anterior",C.sub);return end
 local target=edit.userId
 local snapshot={};for k,v in pairs(patch) do snapshot[k]=v end
 savingTag=true;saveTag.Text="Salvando..."
 tagReadSeq+=1
 task.spawn(function()
  local data,err=api("/admin/player/set",{issuerId=tostring(player.UserId),userId=target,patch=snapshot})
  savingTag=false
  if not alive then return end
  saveTag.Text="Salvar no banco"
  if data and type(data.user)=="table" and tostring(data.user.userId)==target then
   if activeUsers[target] then for k,v in pairs(data.user) do activeUsers[target][k]=v end end
   if requestedEditorId==target and edit.userId==target then fillEditor(data.user) end
   refreshList();renderOnline();setStatus(okMsg or "salvo no banco",C.good);loadTagList()
  else setStatus("erro: "..tostring(err or "perfil não confirmado"),C.bad) end
 end)
end

saveTag.MouseButton1Click:Connect(function()
	local patch = {Tag = tagName.Text, TagEffect = edit.effect, TagFont = edit.font, Active = edit.Active, blacklisted = edit.blacklisted}
	if cor1Box.Text:gsub("%s", "") ~= "" then
		if not parseColor(cor1Box.Text) then setStatus("cor 1 invalida (use 255,0,0 ou #ff0000)", C.bad); return end
		patch.Cor1Tag = cor1Box.Text
	end
	if cor2Box.Text:gsub("%s", "") ~= "" then
		if not parseColor(cor2Box.Text) then setStatus("cor 2 invalida (use 255,0,0 ou #ff0000)", C.bad); return end
		patch.Cor2Tag = cor2Box.Text
	end
	saveTagPatch(patch, "tag salva: " .. tagName.Text)
end)
plusBtn.MouseButton1Click:Connect(function() saveTagPatch({Plus = not edit.Plus}, (not edit.Plus) and "Plus dado" or "Plus removido") end)
coinBtn.MouseButton1Click:Connect(function() saveTagPatch({CoinCliker = not edit.CoinCliker}, (not edit.CoinCliker) and "CoinCliker dado" or "CoinCliker removido") end)
activeBtn.MouseButton1Click:Connect(function() edit.Active = not edit.Active; refreshEditor() end)
banBtn.MouseButton1Click:Connect(function() edit.blacklisted = not edit.blacklisted; refreshEditor() end)
local resetArm = 0
resetTag.MouseButton1Click:Connect(function()
	if os.clock() - resetArm > 3 then resetArm = os.clock(); resetTag.Text = "Confirmar?"; return end
	resetArm = 0; resetTag.Text = "Resetar tag"
	saveTagPatch({reset = true}, "tag resetada pro padrao")
end)
reloadTag.MouseButton1Click:Connect(function() if edit.userId then loadTagUser(edit.userId) end end)
tagRefresh.MouseButton1Click:Connect(loadTagList)
tagSearch.FocusLost:Connect(function(enter) if enter then loadTagList() end end)
tagUseSel.MouseButton1Click:Connect(function()
	for id in pairs(selected) do loadTagUser(id); return end
	if tagSearch.Text:match("^%d+$") then loadTagUser(tagSearch.Text) else setStatus("selecione 1 pessoa na lista da esquerda", C.bad) end
end)


do
 local row=hrow(12,30)
 local giveAura=button(row,"Doar aura",1,UDim2.new(0.5,-3,1,0))
 local giveTheme=button(row,"Doar Singularidade",2,UDim2.new(0.5,-3,1,0));giveTheme.TextSize=11
 local grants=new("Frame",{Size=UDim2.new(1,-24,1,-56),Position=UDim2.fromOffset(12,44),BackgroundColor3=C.bg,Visible=false,ZIndex=30},frame)
 local close=button(grants,"×",1,UDim2.fromOffset(30,30));close.Position=UDim2.new(1,-36,0,4)
 close.MouseButton1Click:Connect(function() grants.Visible=false end)
 local list=new("ScrollingFrame",{Size=UDim2.new(1,-24,1,-48),Position=UDim2.fromOffset(12,40),BackgroundTransparency=1,AutomaticCanvasSize=Enum.AutomaticSize.Y,CanvasSize=UDim2.new(),ScrollBarThickness=4},grants)
 new("UIListLayout",{Padding=UDim.new(0,6)},list)
 local function donate(kind,name)
  if not edit.userId then setStatus("escolha alguém primeiro",C.bad);return end
  local target=edit.userId
  task.spawn(function()
   local data,err=api("/admin/entitlement",{issuerId=tostring(player.UserId),userId=target,kind=kind,name=name,enabled=true})
   if data then setStatus("Doação permanente salva: "..name,C.good);grants.Visible=false;loadTagUser(target)
   else setStatus(tostring(err),C.bad) end
  end)
 end
 for i,v in ipairs({{"angel","Arcanjo"},{"demon","Demon"},{"cherry","Cherry"},{"blessed","Blessed"},{"tech","Tech"},{"piano","Pianist"},{"aura","Empty"},{"void","Void"}}) do
  local b=button(list,"Doar "..v[2],i,UDim2.new(1,0,0,32));b.MouseButton1Click:Connect(function() donate("auras",v[1]) end)
 end
 giveAura.MouseButton1Click:Connect(function() if edit.userId then grants.Visible=true else setStatus("escolha alguém primeiro",C.bad) end end)
 giveTheme.MouseButton1Click:Connect(function() donate("themes","Singularity") end)
end


picker = new("Frame", {Size = UDim2.new(1, -24, 1, -56), Position = UDim2.fromOffset(12, 44), BackgroundColor3 = C.bg, Visible = false, ZIndex = 20}, frame)
corner(picker, 10); stroke(picker, C.accent)
local pickTitle = new("TextLabel", {Size = UDim2.new(1, -60, 0, 30), Position = UDim2.fromOffset(12, 4), BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 14, TextColor3 = C.accent, TextXAlignment = Enum.TextXAlignment.Left, Text = ""}, picker)
local pickClose = button(picker, "×", 0, UDim2.fromOffset(28, 28)); pickClose.Position = UDim2.new(1, -36, 0, 5); pickClose.TextSize = 20
local pickList = new("ScrollingFrame", {Size = UDim2.new(1, -24, 1, -46), Position = UDim2.fromOffset(12, 38), BackgroundTransparency = 1, BorderSizePixel = 0, ScrollBarThickness = 3, CanvasSize = UDim2.new(), AutomaticCanvasSize = Enum.AutomaticSize.Y}, picker)
new("UIGridLayout", {CellSize = UDim2.new(0.5, -4, 0, 40), CellPadding = UDim2.fromOffset(6, 6), SortOrder = Enum.SortOrder.LayoutOrder}, pickList)
pickClose.MouseButton1Click:Connect(function() picker.Visible = false end)
local function openPicker(kind)
	for _, c in ipairs(pickList:GetChildren()) do if c:IsA("GuiObject") then c:Destroy() end end
	if kind == "effect" then
		pickTitle.Text = "Escolha o efeito da tag"
		for i, e in ipairs(TAG_EFFECTS) do
			local b = button(pickList, "", i, UDim2.new())
			if edit.effect == e[1] then setOn(b, true) end
			local tagColor = e[4]:find("ex") and C.plus or (e[4] == "aura" and C.good or C.accent)
			new("TextLabel", {Size = UDim2.new(1, -12, 0, 18), Position = UDim2.fromOffset(8, 3), BackgroundTransparency = 1, Font = Enum.Font.GothamBold, TextSize = 13, TextColor3 = edit.effect == e[1] and C.ink or C.text, TextXAlignment = Enum.TextXAlignment.Left, Text = e[2] .. "  (" .. e[1] .. ")" .. (e[4] ~= "" and ("  · " .. (e[4]:find("ex") and "EXCLUSIVA" or "") .. (e[4]:find("aura") and (e[4]:find("ex") and " + AURA" or "AURA") or "")) or "")}, b)
			new("TextLabel", {Size = UDim2.new(1, -12, 0, 14), Position = UDim2.fromOffset(8, 21), BackgroundTransparency = 1, Font = Enum.Font.Gotham, TextSize = 11, TextColor3 = edit.effect == e[1] and C.ink or tagColor, TextXAlignment = Enum.TextXAlignment.Left, TextTruncate = Enum.TextTruncate.AtEnd, Text = e[3]}, b)
			b.MouseButton1Click:Connect(function() edit.effect = e[1]; refreshEditor(); picker.Visible = false end)
		end
	else
		pickTitle.Text = "Escolha a fonte"
		for i, f in ipairs(TAG_FONTS) do
			local b = button(pickList, f, i, UDim2.new())
			pcall(function() b.FontFace = Font.fromEnum(Enum.Font[f]) end)
			b.TextSize = 16
			if edit.font == f then setOn(b, true) end
			b.MouseButton1Click:Connect(function() edit.font = f; refreshEditor(); picker.Visible = false end)
		end
	end
	picker.Visible = true
end
effectBtn.MouseButton1Click:Connect(function() openPicker("effect") end)
fontBtn.MouseButton1Click:Connect(function() openPicker("font") end)
refreshPalTarget(); refreshEditor()
end
buildTagEditor()

local remPane = pane(); remPane.Visible = false
local function selectTab(which)
 commandsPane.Visible = which == "commands"; messagePane.Visible = which == "message"; online.Visible = which == "users"; chatPane.Visible = which == "chat"; remPane.Visible = which == "remotes"; tagsPane.Visible = which == "tags"
 if which ~= "tags" then picker.Visible = false end
 setOn(cmdTab, which == "commands"); setOn(msgTab, which == "message"); setOn(usersTab, which == "users"); setOn(chatTab, which == "chat"); setOn(remTab, which == "remotes"); setOn(tagsTab, which == "tags")
 if tagsPane.Visible and #tagResults:GetChildren() <= 2 then loadTagList() end
 if chatPane.Visible then task.spawn(chatSync) end
 if online.Visible then loadOnline() end
end
cmdTab.MouseButton1Click:Connect(function() selectTab("commands") end)
msgTab.MouseButton1Click:Connect(function() selectTab("message") end)
chatTab.MouseButton1Click:Connect(function() selectTab("chat") end)
onlineBtn.MouseButton1Click:Connect(function() selectTab("users") end)
tagsTab.MouseButton1Click:Connect(function() selectTab("tags") end)
openTagEditor = function(id) selectTab("tags"); loadTagUser(id) end
oClose.MouseButton1Click:Connect(function() selectTab("commands") end)
selectTab("commands")
oServer.MouseButton1Click:Connect(function() onlineScope = "server"; loadOnline() end)
oAll.MouseButton1Click:Connect(function() onlineScope = "all"; loadOnline() end)
oRefresh.MouseButton1Click:Connect(loadOnline)
task.spawn(function()
	while alive do
		task.wait(10)
		if alive and frame.Visible and online.Visible then loadOnline() end
	end
end)


local function parseChatTarget(rest)
	local token, after = rest:match("^@(%S+)%s*(.*)$")
	if not token then return nil, rest end
	local t = token:lower()
	if t == "all" or t == "todos" then return "all", after end
	if t == "me" or t == "eu" then return {tostring(player.UserId)}, after end
	local found = findPlayers(token)
	if #found == 0 then return false, after end
	local ids = {}
	for _, p in ipairs(found) do table.insert(ids, tostring(p.userId)) end
	return ids, after
end

local function onChat(text)
	if type(text) ~= "string" or text:sub(1, #PREFIX) ~= PREFIX then return end
	local name, rest = text:sub(#PREFIX + 1):match("^(%S+)%s*(.*)$")
	local def = name and byName[name:lower()]
	if not def then return end
	if def.name == "announce" then
		if rest ~= "" then run(def, rest, "all", "server") end
		return
	end
	local targets, after = parseChatTarget(rest or "")
	if targets == false then setStatus("jogador não encontrado", C.bad); return end
	run(def, after, targets)
end

local conns = {}
startupCheckpoint("eventos do chat")
if TextChatService.ChatVersion == Enum.ChatVersion.TextChatService then
	table.insert(conns, TextChatService.SendingMessage:Connect(function(msg) onChat(msg.Text) end))
else
	table.insert(conns, player.Chatted:Connect(onChat))
end
task.spawn(function()
	while alive do
		task.wait(8)
		if alive then loadTargets() end
	end
end)

task.spawn(function()
 while alive do
  task.wait(1)
  if alive then
   for id,rec in pairs(targetRows) do if frame.Visible and rec.row.Visible then setChanged(rec.duration,"Text",executionText(id)) end end
   for id,rec in pairs(onlineRows) do if online.Visible then setChanged(rec.duration,"Text",executionText(id)) end end
   if phoneGui.Visible and phoneFor then setChanged(phoneDuration,"Text",executionText(phoneFor)) end
  end
 end
end)
refreshTargets(); refreshList(); refreshScope()
setStatus("digite a senha para conectar",C.sub)

do
 local connectionBar=new("Frame",{Size=UDim2.new(1,-24,0,64),Position=UDim2.fromOffset(12,405),BackgroundTransparency=1},frame)
 local passwordBox=box(connectionBar,"Senha de administrador",1,30)
 passwordBox.Size=UDim2.new(1,-130,0,30)
 local enter=button(connectionBar,"Conectar",2,UDim2.fromOffset(120,30));enter.Position=UDim2.new(1,-120,0,0)
 local feedback=new("TextLabel",{Size=UDim2.new(1,0,0,28),Position=UDim2.fromOffset(0,34),BackgroundTransparency=1,Text="Desconectado",TextColor3=C.sub,TextSize=11,TextWrapped=true,TextXAlignment=Enum.TextXAlignment.Left},connectionBar)
 local requestSeq=0
 passwordBox.Text=ADMIN_KEY
 reportApiError=function(err)
  local message=tostring(err or "falha na API")
  feedback.Text=message;feedback.TextColor3=C.bad
  setStatus(message,C.bad);dot.BackgroundColor3=C.bad
  if message=="senha recusada pela API" or message=="acesso negado: confira a senha e o proprietário configurado na API" then
   connected=false;ADMIN_KEY="";enter.Text="Reconectar"
  end
 end
 local function submit()
  if connecting then return end
  if passwordBox.Text=="" and ADMIN_KEY=="" then reportApiError("Digite sua senha.");return end
  if type(httpRequest)~="function" then reportApiError("Seu ambiente não oferece pedidos HTTP.");return end
  if passwordBox.Text~="" then ADMIN_KEY=passwordBox.Text end
  passwordBox.Text=""
  connected=false;connecting=true;enter.Text="Conectando..."
  feedback.Text="Conectando à API...";feedback.TextColor3=C.sub;dot.BackgroundColor3=C.sub
  requestSeq+=1
  local seq=requestSeq
  task.spawn(function()
   local ok,data,err=pcall(function() return apiInner("/admin/active",{issuerId=tostring(player.UserId),jobId=game.JobId,scope="all"},true) end)
   if not alive or requestSeq~=seq then return end
   connecting=false
   if not ok then data=nil;err="Falha ao verificar acesso. Tente novamente." end
   if data then
    connected=true;enter.Text="Reconectar";feedback.Text="Conectado à API";feedback.TextColor3=C.good;dot.BackgroundColor3=C.good
    setStatus("acesso confirmado",C.good);loadTargets()
   else
    connected=false;enter.Text="Tentar novamente";reportApiError(err)
   end
  end)
 end
 enter.MouseButton1Click:Connect(submit)
 passwordBox.FocusLost:Connect(function(pressed) if pressed then submit() end end)
 dot.BackgroundColor3=C.sub
 startupCheckpoint("painel completo")
 uiReady=true
 if ADMIN_KEY~="" then task.defer(submit) end
end


env.VHUB_ADMIN_SHUTDOWN = function()
	alive = false
 if type(env.VHUB_ADMIN_LOGIN_CLOSE)=="function" then pcall(env.VHUB_ADMIN_LOGIN_CLOSE) end
	if tpFailConn then pcall(function() tpFailConn:Disconnect() end) tpFailConn = nil end
	for _, c in ipairs(conns) do pcall(function() c:Disconnect() end) end
	if cameraConn then cameraConn:Disconnect() end
	pcall(function() gui:Destroy() end)
end
end
task.delay(8,function()
 if not startupFinished and startupText and startupText.Parent then
  startupText.Text=startupResult(false,startupStage)
 end
end)
local ok=pcall(runPanel)
startupFinished=true
if ok then
 if startupGui then startupGui:Destroy() end
else
 if startupText and startupText.Parent then startupText.Text=startupResult(false,startupStage) end
 warn("VERCEL: falha na abertura em "..startupStage)
end
]====]
local panel,message=compilePanel(loadstring,panelSource)
if not panel then text.Text=message;warn("Vercel: "..message);return end
local ok=pcall(panel)
if ok then gui:Destroy() else text.Text="Vercel: o código foi aceito, mas houve falha ao iniciar. Envie uma foto." end
