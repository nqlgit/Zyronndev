--[[
    Zyron Hub v2.0.0
    The Mimic — ESP, Walkspeed, Noclip, Auto Carnival
    by ZYRONNDEV
]]

local Players          = game:GetService("Players")
local TweenService     = game:GetService("TweenService")
local RunService       = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer      = Players.LocalPlayer
local PlayerGui        = LocalPlayer:WaitForChild("PlayerGui")

-- Evita duplicatas
local oldGui = PlayerGui:FindFirstChild("ZyronndevHub")
if oldGui then oldGui:Destroy() end

-- ══════════════════════════════════════
--           LOADING SCREEN
-- ══════════════════════════════════════
local loadGui = Instance.new("ScreenGui")
loadGui.Name = "ZyronndevLoading"
loadGui.ResetOnSpawn = false
loadGui.DisplayOrder = 99
loadGui.Parent = PlayerGui

local bg = Instance.new("Frame")
bg.Size = UDim2.new(1,0,1,0)
bg.BackgroundColor3 = Color3.fromRGB(5,5,10)
bg.BorderSizePixel = 0
bg.Parent = loadGui

local titleLbl = Instance.new("TextLabel")
titleLbl.Size = UDim2.new(0,400,0,60)
titleLbl.Position = UDim2.new(0.5,-200,0.35,0)
titleLbl.BackgroundTransparency = 1
titleLbl.Text = "ZYRON HUB"
titleLbl.TextColor3 = Color3.fromRGB(220,30,30)
titleLbl.TextScaled = true
titleLbl.Font = Enum.Font.GothamBold
titleLbl.TextStrokeTransparency = 0.4
titleLbl.Parent = bg

local subLbl = Instance.new("TextLabel")
subLbl.Size = UDim2.new(0,400,0,30)
subLbl.Position = UDim2.new(0.5,-200,0.45,0)
subLbl.BackgroundTransparency = 1
subLbl.Text = "Carregando script..."
subLbl.TextColor3 = Color3.fromRGB(180,180,180)
subLbl.TextScaled = true
subLbl.Font = Enum.Font.Gotham
subLbl.Parent = bg

local barBg = Instance.new("Frame")
barBg.Size = UDim2.new(0,400,0,10)
barBg.Position = UDim2.new(0.5,-200,0.55,0)
barBg.BackgroundColor3 = Color3.fromRGB(30,30,30)
barBg.BorderSizePixel = 0
barBg.Parent = bg
Instance.new("UICorner", barBg).CornerRadius = UDim.new(1,0)

local bar = Instance.new("Frame")
bar.Size = UDim2.new(0,0,1,0)
bar.BackgroundColor3 = Color3.fromRGB(220,30,30)
bar.BorderSizePixel = 0
bar.Parent = barBg
Instance.new("UICorner", bar).CornerRadius = UDim.new(1,0)

local statusLbl = Instance.new("TextLabel")
statusLbl.Size = UDim2.new(0,400,0,25)
statusLbl.Position = UDim2.new(0.5,-200,0.59,0)
statusLbl.BackgroundTransparency = 1
statusLbl.Text = "Inicializando..."
statusLbl.TextColor3 = Color3.fromRGB(120,120,120)
statusLbl.TextScaled = true
statusLbl.Font = Enum.Font.Gotham
statusLbl.Parent = bg

local function setProgress(pct, msg)
    TweenService:Create(bar, TweenInfo.new(0.3), {Size=UDim2.new(pct,0,1,0)}):Play()
    statusLbl.Text = msg
    task.wait(0.4)
end

setProgress(0.2,  "Conectando...")
setProgress(0.4,  "Carregando ESP...")
setProgress(0.6,  "Montando painel...")
setProgress(0.8,  "Configurando carnival...")
setProgress(1.0,  "Pronto!")
task.wait(0.4)
loadGui:Destroy()

-- ══════════════════════════════════════
--              CONFIG
-- ══════════════════════════════════════
local Camera    = workspace.CurrentCamera
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

local Config = {
    WalkSpeed     = 32,
    JumpPower     = 100,
    NoclipEnabled = false,
    ESPEntity     = true,
    ESPItems      = true,
    ESPPlayers    = true,
    AutoCarnival  = false,
    ESPColor      = Color3.fromRGB(255,60,60),
    PlayerColor   = Color3.fromRGB(60,200,255),
    ItemColor     = Color3.fromRGB(255,220,0),
}

local function applyStats()
    local h = Character and Character:FindFirstChildOfClass("Humanoid")
    if h then h.WalkSpeed=Config.WalkSpeed; h.JumpPower=Config.JumpPower end
end

LocalPlayer.CharacterAdded:Connect(function(char)
    Character = char; char:WaitForChild("Humanoid"); applyStats()
end)
applyStats()

-- ══════════════════════════════════════
--              NOCLIP
-- ══════════════════════════════════════
RunService.Stepped:Connect(function()
    if not Config.NoclipEnabled or not Character then return end
    for _, p in ipairs(Character:GetDescendants()) do
        if p:IsA("BasePart") then p.CanCollide = false end
    end
end)

-- ══════════════════════════════════════
--           ESP HELPER
-- ══════════════════════════════════════
local ESPObjects = {}

local function createESP(part, color, label)
    local box = Instance.new("SelectionBox")
    box.Color3=color; box.LineThickness=0.05
    box.SurfaceTransparency=0.7; box.SurfaceColor3=color
    box.Adornee=part; box.Parent=Camera

    local bill = Instance.new("BillboardGui")
    bill.Size=UDim2.new(0,120,0,30); bill.StudsOffset=Vector3.new(0,2,0)
    bill.AlwaysOnTop=true; bill.Adornee=part; bill.Parent=Camera

    local lbl = Instance.new("TextLabel")
    lbl.Size=UDim2.new(1,0,1,0); lbl.BackgroundTransparency=1
    lbl.TextColor3=color; lbl.TextStrokeTransparency=0
    lbl.TextScaled=true; lbl.Font=Enum.Font.GothamBold
    lbl.Text=label; lbl.Parent=bill

    return {box=box, bill=bill}
end

local function removeESP(key)
    if not ESPObjects[key] then return end
    if ESPObjects[key].box  then ESPObjects[key].box:Destroy()  end
    if ESPObjects[key].bill then ESPObjects[key].bill:Destroy() end
    ESPObjects[key] = nil
end

-- ══════════════════════════════════════
--           PLAYER ESP
-- ══════════════════════════════════════
local function setupPlayerESP(player)
    if player == LocalPlayer then return end
    player.CharacterAdded:Connect(function(char)
        local root = char:WaitForChild("HumanoidRootPart", 5)
        if root and Config.ESPPlayers then
            removeESP("pl_"..player.UserId)
            ESPObjects["pl_"..player.UserId] = createESP(root, Config.PlayerColor, player.Name)
        end
    end)
    if player.Character then
        local root = player.Character:FindFirstChild("HumanoidRootPart")
        if root and Config.ESPPlayers then
            ESPObjects["pl_"..player.UserId] = createESP(root, Config.PlayerColor, player.Name)
        end
    end
end

Players.PlayerAdded:Connect(setupPlayerESP)
Players.PlayerRemoving:Connect(function(p) removeESP("pl_"..p.UserId) end)
for _, p in ipairs(Players:GetPlayers()) do setupPlayerESP(p) end

-- ══════════════════════════════════════
--       ENTITY + ITEM ESP
-- ══════════════════════════════════════
local EntityKW  = {"Mimic","Kuchisake","Shako","Spirit","Ghost","Monster","Enemy","Creature"}
local ItemKW    = {"Lantern0","Candle0","Paper","Object0"}
local Blacklist = {"Floor","Wall","Ceiling","Mesh","Modular","Terrain","Baseplate","Union","Wedge","Truss"}

local function blocked(name)
    local l = name:lower()
    for _, b in ipairs(Blacklist) do if l:find(b:lower()) then return true end end
    return false
end

local function match(name, list)
    local l = name:lower()
    for _, kw in ipairs(list) do if l:find(kw:lower()) then return true end end
    return false
end

local scanned = {}

local function scanWorld()
    for _, obj in ipairs(workspace:GetDescendants()) do
        local ptr = tostring(obj)
        if scanned[ptr] or blocked(obj.Name) then continue end
        if obj:IsA("Model") then
            local root = obj:FindFirstChild("HumanoidRootPart") or obj:FindFirstChildOfClass("BasePart")
            if root then
                if Config.ESPEntity and match(obj.Name, EntityKW) then
                    ESPObjects["e_"..ptr] = createESP(root, Config.ESPColor, "⚠ "..obj.Name)
                    scanned[ptr] = true
                elseif Config.ESPItems and match(obj.Name, ItemKW) then
                    ESPObjects["i_"..ptr] = createESP(root, Config.ItemColor, "🔑 "..obj.Name)
                    scanned[ptr] = true
                end
            end
        elseif obj:IsA("BasePart") and Config.ESPItems and match(obj.Name, ItemKW) then
            ESPObjects["i_"..ptr] = createESP(obj, Config.ItemColor, "🔑 "..obj.Name)
            scanned[ptr] = true
        end
    end
end

workspace.DescendantRemoving:Connect(function(obj)
    local ptr = tostring(obj)
    scanned[ptr] = nil
    removeESP("e_"..ptr); removeESP("i_"..ptr)
end)

task.spawn(function()
    while true do scanWorld(); task.wait(2) end
end)

-- ══════════════════════════════════════
--         AUTO CARNIVAL
-- ══════════════════════════════════════
local CarnivalKW = {"carnival","game","minigame","play","booth","start"}

task.spawn(function()
    while true do
        if Config.AutoCarnival then
            local root = Character and Character:FindFirstChild("HumanoidRootPart")
            if root then
                for _, obj in ipairs(workspace:GetDescendants()) do
                    local name = (obj.Parent and obj.Parent.Name or ""):lower()
                    if obj:IsA("ProximityPrompt") or obj:IsA("ClickDetector") then
                        local part = obj.Parent
                        local near = part and part:IsA("BasePart") and
                            (root.Position - part.Position).Magnitude < 25
                        local action = obj:IsA("ProximityPrompt") and (obj.ActionText or ""):lower() or name
                        local isCarnival = false
                        for _, kw in ipairs(CarnivalKW) do
                            if action:find(kw) or name:find(kw) then isCarnival=true; break end
                        end
                        if isCarnival and near then
                            pcall(function()
                                if obj:IsA("ProximityPrompt") then fireproximityprompt(obj)
                                else fireclickdetector(obj) end
                            end)
                        end
                    end
                end
            end
        end
        task.wait(0.5)
    end
end)

-- ══════════════════════════════════════
--         CORES / UI BASE
-- ══════════════════════════════════════
local COLORS = {
    background = Color3.fromRGB(15,16,19),
    panel      = Color3.fromRGB(23,24,29),
    surface    = Color3.fromRGB(32,34,40),
    hover      = Color3.fromRGB(44,46,53),
    border     = Color3.fromRGB(57,59,67),
    text       = Color3.fromRGB(242,243,245),
    muted      = Color3.fromRGB(155,158,168),
    accent     = Color3.fromRGB(220,30,30),
    success    = Color3.fromRGB(130,205,155),
}

local function create(className, props, parent)
    local obj = Instance.new(className)
    for k,v in pairs(props or {}) do obj[k]=v end
    obj.Parent = parent
    return obj
end

local function round(obj, r)
    create("UICorner", {CornerRadius=UDim.new(0,r)}, obj)
end

local function outline(obj, color, t)
    create("UIStroke", {Color=color or COLORS.border, Thickness=1, Transparency=t or 0}, obj)
end

-- ══════════════════════════════════════
--           INTERFACE PRINCIPAL
-- ══════════════════════════════════════
local gui = create("ScreenGui", {
    Name="ZyronndevHub",
    ResetOnSpawn=false,
    IgnoreGuiInset=false,
    DisplayOrder=20,
    ZIndexBehavior=Enum.ZIndexBehavior.Sibling,
}, PlayerGui)

local main = create("Frame", {
    Name="Main",
    Size=UDim2.fromOffset(570,390),
    Position=UDim2.new(0.5,-285,0.5,-195),
    BackgroundColor3=COLORS.background,
    BorderSizePixel=0,
    ClipsDescendants=true,
}, gui)
round(main,12); outline(main, COLORS.border)

-- Topbar
local topbar = create("Frame", {
    Size=UDim2.new(1,0,0,53),
    BackgroundColor3=COLORS.panel,
    BorderSizePixel=0,
}, main)

local logo = create("Frame", {
    Position=UDim2.fromOffset(14,11),
    Size=UDim2.fromOffset(31,31),
    BackgroundColor3=COLORS.surface,
    BorderSizePixel=0,
}, topbar)
round(logo,8)

create("TextLabel", {
    Size=UDim2.fromScale(1,1),
    BackgroundTransparency=1,
    Text="Z",
    TextColor3=COLORS.text,
    Font=Enum.Font.GothamBold,
    TextSize=20,
}, logo)

create("TextLabel", {
    Position=UDim2.fromOffset(54,8),
    Size=UDim2.new(0,220,0,22),
    BackgroundTransparency=1,
    Text="Zyron Hub",
    TextColor3=COLORS.text,
    Font=Enum.Font.GothamBold,
    TextSize=17,
    TextXAlignment=Enum.TextXAlignment.Left,
}, topbar)

create("TextLabel", {
    Position=UDim2.fromOffset(55,29),
    Size=UDim2.new(0,220,0,15),
    BackgroundTransparency=1,
    Text="THE MIMIC  /  v2.0.0",
    TextColor3=COLORS.muted,
    Font=Enum.Font.Gotham,
    TextSize=9,
    TextXAlignment=Enum.TextXAlignment.Left,
}, topbar)

local function makeBtn(parent, text, size, pos)
    local btn = create("TextButton", {
        Size=size, Position=pos,
        BackgroundColor3=COLORS.surface,
        Text=text, TextColor3=COLORS.text,
        Font=Enum.Font.GothamMedium, TextSize=12,
        BorderSizePixel=0, AutoButtonColor=false,
    }, parent)
    round(btn,7)
    btn.MouseEnter:Connect(function()
        TweenService:Create(btn,TweenInfo.new(0.12),{BackgroundColor3=COLORS.hover}):Play()
    end)
    btn.MouseLeave:Connect(function()
        TweenService:Create(btn,TweenInfo.new(0.12),{BackgroundColor3=COLORS.surface}):Play()
    end)
    return btn
end

local closeBtn    = makeBtn(topbar,"×",UDim2.fromOffset(34,30),UDim2.new(1,-44,0,11))
closeBtn.TextSize = 22
local minimizeBtn = makeBtn(topbar,"—",UDim2.fromOffset(34,30),UDim2.new(1,-84,0,11))

-- Sidebar
local sidebar = create("Frame", {
    Position=UDim2.fromOffset(0,53),
    Size=UDim2.new(0,155,1,-53),
    BackgroundColor3=COLORS.panel,
    BorderSizePixel=0,
}, main)

create("TextLabel", {
    Position=UDim2.fromOffset(16,16),
    Size=UDim2.new(1,-25,0,20),
    BackgroundTransparency=1,
    Text="MENU",
    TextColor3=COLORS.muted,
    Font=Enum.Font.GothamBold,
    TextSize=10,
    TextXAlignment=Enum.TextXAlignment.Left,
}, sidebar)

local nav = create("Frame", {
    Position=UDim2.fromOffset(10,44),
    Size=UDim2.new(1,-20,0,200),
    BackgroundTransparency=1,
}, sidebar)
create("UIListLayout",{Padding=UDim.new(0,7),SortOrder=Enum.SortOrder.LayoutOrder},nav)

-- Conteúdo
local content = create("Frame",{
    Name="Content",
    Position=UDim2.fromOffset(155,53),
    Size=UDim2.new(1,-155,1,-53),
    BackgroundTransparency=1,
}, main)

local pages, navButtons = {}, {}

local function makePage(name)
    local page = create("ScrollingFrame",{
        Name=name,
        Size=UDim2.new(1,-24,1,-24),
        Position=UDim2.fromOffset(12,12),
        BackgroundTransparency=1,
        BorderSizePixel=0,
        ScrollBarThickness=3,
        ScrollBarImageColor3=COLORS.border,
        CanvasSize=UDim2.new(0,0,0,0),
        AutomaticCanvasSize=Enum.AutomaticSize.Y,
        Visible=false,
    }, content)
    create("UIListLayout",{Padding=UDim.new(0,10),SortOrder=Enum.SortOrder.LayoutOrder},page)
    pages[name] = page
    return page
end

local function heading(parent, title, subtitle)
    local c = create("Frame",{Size=UDim2.new(1,0,0,55),BackgroundTransparency=1},parent)
    create("TextLabel",{Size=UDim2.new(1,0,0,27),BackgroundTransparency=1,Text=title,
        TextColor3=COLORS.text,Font=Enum.Font.GothamBold,TextSize=20,
        TextXAlignment=Enum.TextXAlignment.Left},c)
    create("TextLabel",{Position=UDim2.fromOffset(0,29),Size=UDim2.new(1,0,0,20),
        BackgroundTransparency=1,Text=subtitle,TextColor3=COLORS.muted,
        Font=Enum.Font.Gotham,TextSize=11,TextXAlignment=Enum.TextXAlignment.Left},c)
end

-- Notificações
local notif = create("TextLabel",{
    AnchorPoint=Vector2.new(1,1),
    Position=UDim2.new(1,-15,1,-15),
    Size=UDim2.fromOffset(250,42),
    BackgroundColor3=COLORS.surface,
    TextColor3=COLORS.text,
    Text="", TextSize=12,
    Font=Enum.Font.GothamMedium,
    TextWrapped=true, Visible=false, ZIndex=10,
}, gui)
round(notif,8); outline(notif,COLORS.border)

local notifId = 0
local function notify(msg)
    notifId += 1
    local id = notifId
    notif.Text = "  "..msg
    notif.Visible = true
    task.delay(2.5, function()
        if notifId == id then notif.Visible = false end
    end)
end

-- ══════════════════════════════════════
--    TOGGLE PILL (função reutilizável)
-- ══════════════════════════════════════
local function createToggle(parent, labelText, default, callback)
    local row = create("Frame",{
        Size=UDim2.new(1,0,0,42),
        BackgroundColor3=COLORS.panel,
        BorderSizePixel=0,
    }, parent)
    round(row,9); outline(row,COLORS.border)

    create("TextLabel",{
        Position=UDim2.fromOffset(14,0),
        Size=UDim2.new(0.65,0,1,0),
        BackgroundTransparency=1,
        Text=labelText,
        TextColor3=COLORS.text,
        Font=Enum.Font.Gotham,
        TextSize=12,
        TextXAlignment=Enum.TextXAlignment.Left,
    }, row)

    local pill = create("Frame",{
        Size=UDim2.fromOffset(46,24),
        Position=UDim2.new(1,-58,0.5,-12),
        BackgroundColor3=default and COLORS.accent or COLORS.surface,
        BorderSizePixel=0,
    }, row)
    round(pill,50)
    outline(pill,COLORS.border)

    local circle = create("Frame",{
        Size=UDim2.fromOffset(18,18),
        Position=default and UDim2.new(1,-21,0.5,-9) or UDim2.new(0,3,0.5,-9),
        BackgroundColor3=COLORS.text,
        BorderSizePixel=0,
    }, pill)
    round(circle,50)

    local state = default
    local db = false

    local hitbox = create("TextButton",{
        Size=UDim2.new(1,0,1,0),
        BackgroundTransparency=1,
        Text="",
    }, row)

    hitbox.MouseButton1Click:Connect(function()
        if db then return end; db=true
        state = not state
        TweenService:Create(pill,TweenInfo.new(0.2),{
            BackgroundColor3=state and COLORS.accent or COLORS.surface
        }):Play()
        TweenService:Create(circle,TweenInfo.new(0.2),{
            Position=state and UDim2.new(1,-21,0.5,-9) or UDim2.new(0,3,0.5,-9)
        }):Play()
        callback(state)
        task.wait(0.2); db=false
    end)
end

-- ══════════════════════════════════════
--         PÁGINA: COMBAT / ESP
-- ══════════════════════════════════════
local espPage = makePage("ESP")
heading(espPage,"ESP","Visualização de entidades, itens e jogadores.")

createToggle(espPage,"ESP Entidades", Config.ESPEntity, function(v)
    Config.ESPEntity = v
    if not v then for k in pairs(ESPObjects) do
        if k:sub(1,2)=="e_" then removeESP(k) end
    end end
    notify("ESP Entidades: "..(v and "ON" or "OFF"))
end)

createToggle(espPage,"ESP Itens", Config.ESPItems, function(v)
    Config.ESPItems = v
    if not v then for k in pairs(ESPObjects) do
        if k:sub(1,2)=="i_" then removeESP(k) end
    end end
    notify("ESP Itens: "..(v and "ON" or "OFF"))
end)

createToggle(espPage,"ESP Players", Config.ESPPlayers, function(v)
    Config.ESPPlayers = v
    if not v then for k in pairs(ESPObjects) do
        if k:sub(1,3)=="pl_" then removeESP(k) end
    end else
        for _, p in ipairs(Players:GetPlayers()) do setupPlayerESP(p) end
    end
    notify("ESP Players: "..(v and "ON" or "OFF"))
end)

-- ══════════════════════════════════════
--        PÁGINA: MOVEMENT
-- ══════════════════════════════════════
local movePage = makePage("Movement")
heading(movePage,"Movement","Velocidade, pulo e noclip.")

createToggle(movePage,"Walkspeed x2", true, function(v)
    Config.WalkSpeed = v and 32 or 16
    local h = Character and Character:FindFirstChildOfClass("Humanoid")
    if h then h.WalkSpeed=Config.WalkSpeed end
    notify("Walkspeed: "..(v and "32" or "16"))
end)

createToggle(movePage,"Infinite Jump", true, function(v)
    Config.JumpPower = v and 100 or 50
    local h = Character and Character:FindFirstChildOfClass("Humanoid")
    if h then h.JumpPower=Config.JumpPower end
    notify("Jump Power: "..(v and "100" or "50"))
end)

createToggle(movePage,"Noclip", false, function(v)
    Config.NoclipEnabled = v
    notify("Noclip: "..(v and "ON" or "OFF"))
end)

-- ══════════════════════════════════════
--        PÁGINA: CARNIVAL
-- ══════════════════════════════════════
local carnivalPage = makePage("Carnival")
heading(carnivalPage,"Carnival","Auto completar jogos do carnaval.")

createToggle(carnivalPage,"Auto Carnival Games", false, function(v)
    Config.AutoCarnival = v
    notify("Auto Carnival: "..(v and "ON" or "OFF"))
end)

-- Info card
