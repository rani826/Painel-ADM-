script aqui
-- love_painel❤️ - Rayfield UI - Local Lua (KRNL)
-- Apenas para testes.
-- Premium Key: "lovehubpremium" (salva para não pedir mais)
-- Temp Key (24h): "iloveyouu"

-- Nota: este script tenta carregar a Rayfield via HttpGet. Seu executor deve permitir HttpGet (KRNL normalmente permite).
-- Se preferir, você pode substituir a URL pela sua cópia local da Rayfield.lua.

-- == Configuração de Keys / arquivos ==
local PREMIUM_KEY = "lovehubpremium"
local TEMP_KEY = "iloveyouu"
local TEMP_DURATION = 24 * 60 * 60
local DATA_FOLDER = "love_painel_data"
local PREMIUM_FILE = DATA_FOLDER.."/premium.enabled"
local TEMP_FILE = DATA_FOLDER.."/temp.expire"

-- Criar pasta se executor suportar
if makefolder and not isfolder(DATA_FOLDER) then pcall(function() makefolder(DATA_FOLDER) end) end
local function saveFile(path, content) if writefile then pcall(function() writefile(path, content) end) end end
local function loadFile(path) if isfile and readfile and isfile(path) then local ok, val = pcall(function() return readfile(path) end) if ok then return val end end return nil end
local function deleteFile(path) if isfile and delfile and isfile(path) then pcall(function() delfile(path) end) end end
local function isPremium() if isfile and isfile(PREMIUM_FILE) then return true end return false end
local function isTempValid() local s = loadFile(TEMP_FILE) if s then local ts = tonumber(s) if ts and ts > os.time() then return true end end return false end

local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local StarterGui = game:GetService("StarterGui")

local function notify(msg)
    pcall(function()
        StarterGui:SetCore("SendNotification", {Title = "love painel", Text = msg, Duration = 3})
    end)
end

-- == Load Rayfield ==
local success, Rayfield = pcall(function()
    local url = "https://raw.githubusercontent.com/RayfieldOrg/Rayfield/main/source.lua"
    local str = game:HttpGet(url)
    return loadstring(str)()
end)

if not success or not Rayfield then
    notify("Não foi possível carregar Rayfield. Verifique HttpGet/URL.")
    return
end

-- == Key check UI (pequeno modal) ==
local function acceptKey(k)
    if k == PREMIUM_KEY then
        if makefolder and writefile then saveFile(PREMIUM_FILE, "1") end
        notify("Premium ativado — não pedirá mais keys")
        return true
    elseif k == TEMP_KEY then
        local expire = os.time() + TEMP_DURATION
        if makefolder and writefile then saveFile(TEMP_FILE, tostring(expire)) end
        notify("Key temporária aceita por 24 horas")
        return true
    else
        notify("Key inválida")
        return false
    end
end

if not (isPremium() or isTempValid()) then
    local keyWindow = Rayfield:CreateWindow({
        Name = "Love Painel - Key",
        LoadingTitle = "Love Painel",
        LoadingSubtitle = "Verificando Key",
        ConfigurationSaving = {
            Enabled = false,
        },
    })
    local keySection = keyWindow:CreateSection("Insira sua Key (Premium ou Temporária)")
    local keyInput = keyWindow:CreateInput({
        Name = "Cole a Key",
        PlaceholderText = "Insira a key",
        RemoveTextAfterFocusLost = false,
        Callback = function(value) end
    })
    keyWindow:CreateButton({
        Name = "Usar Key",
        Callback = function()
            local k = keyInput:GetValue()
            if acceptKey(k) then
                keyWindow:Destroy()
            end
        end
    })
    repeat wait() until isPremium() or isTempValid()
end

-- == Main Rayfield UI (compact, móvel, com minimizar) ==
local Window = Rayfield:CreateWindow({
    Name = "love painel ❤️",
    LoadingTitle = "love painel",
    LoadingSubtitle = "Painel ADM - Compacto",
    ConfigurationSaving = {
        Enabled = true,
        FolderName = "LovePainel_Config",
        FileName = "config"
    }
})

local minimized = false
local restoreGui = Instance.new("ScreenGui")
restoreGui.Name = "love_painel_restore"
restoreGui.ResetOnSpawn = false
restoreGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
restoreGui.Enabled = false

local restoreBtn = Instance.new("TextButton")
restoreBtn.Size = UDim2.new(0,140,0,36)
restoreBtn.Position = UDim2.new(0.02,0,0.82,0)
restoreBtn.AnchorPoint = Vector2.new(0,0)
restoreBtn.Text = "Show Love Painel ❤️"
restoreBtn.BackgroundColor3 = Color3.fromRGB(30,30,35)
restoreBtn.TextColor3 = Color3.new(1,1,1)
restoreBtn.Parent = restoreGui

restoreBtn.MouseButton1Click:Connect(function()
    restoreGui.Enabled = false
    minimized = false
    Rayfield:Toggle(true)
end)

local minimizeToggle = Window:CreateToggle({
    Name = "Minimizar (–)",
    CurrentValue = false,
    Flag = "MinimizeFlag",
    Callback = function(val)
        minimized = val
        if minimized then
            Rayfield:Toggle(false)
            restoreGui.Enabled = true
        else
            Rayfield:Toggle(true)
            restoreGui.Enabled = false
        end
    end
})

-- Player dropdown
local playersDropdown = Window:CreateDropdown({
    Name = "Selecionar jogador",
    Options = (function()
        local opts = {}
        for _,p in pairs(Players:GetPlayers()) do table.insert(opts, p.Name) end
        return opts
    end)(),
    CurrentOption = "Selecionar jogador",
    Flag = "PlayerSelect",
    Callback = function(option) end
})

Players.PlayerAdded:Connect(function()
    local arr = {}
    for _,p in pairs(Players:GetPlayers()) do table.insert(arr,p.Name) end
    if Rayfield.Refresh then pcall(function() Rayfield:Refresh("Selecionar jogador", arr) end) end
end)
Players.PlayerRemoving:Connect(function()
    local arr = {}
    for _,p in pairs(Players:GetPlayers()) do table.insert(arr,p.Name) end
    if Rayfield.Refresh then pcall(function() Rayfield:Refresh("Selecionar jogador", arr) end) end
end)

local function getSelectedPlayer()
    local name = Rayfield.Flags and Rayfield.Flags["PlayerSelect"] or ""
    if name == "" or name == "Selecionar jogador" then return nil end
    return Players:FindFirstChild(name)
end

-- Commands
local jailPositions = {}
local frozen = {}
local savedCameraSubject = nil

local function visualKick(target)
    if not target then notify("Selecione jogador válido") return end
    notify("[Visual] Kick: "..target.Name)
end
local function visualBan(target)
    if not target then notify("Selecione jogador válido") return end
    notify("[Visual] Ban: "..target.Name)
end
local function jail(target)
    if not target or not target.Character or not target.Character:FindFirstChild("HumanoidRootPart") then notify("Personagem inválido") return end
    jailPositions[target.Name] = target.Character.HumanoidRootPart.CFrame
    target.Character.HumanoidRootPart.CFrame = CFrame.new(0,200,0)
    notify("Jailed: "..target.Name)
end
local function unjail(target)
    if not target or not target.Character then notify("Personagem inválido") return end
    local prev = jailPositions[target.Name]
    if prev then target.Character.HumanoidRootPart.CFrame = prev jailPositions[target.Name] = nil notify("Unjailed: "..target.Name) else notify("Sem posição salva") end
end
local function freezePlayer(target)
    if not target or not target.Character then notify("Personagem inválido") return end
    local hum = target.Character:FindFirstChildWhichIsA("Humanoid")
    if hum then
        frozen[target.Name] = {WalkSpeed = hum.WalkSpeed or 16, JumpPower = hum.JumpPower or 50}
        hum.WalkSpeed = 0
        pcall(function() hum.JumpPower = 0 end)
        notify("Frozen: "..target.Name)
    end
end
local function unfreezePlayer(target)
    if not target or not target.Character then notify("Personagem inválido") return end
    local hum = target.Character:FindFirstChildWhichIsA("Humanoid")
    local prev = frozen[target.Name]
    if hum and prev then
        pcall(function() hum.WalkSpeed = prev.WalkSpeed end)
        pcall(function() hum.JumpPower = prev.JumpPower end)
        frozen[target.Name] = nil
        notify("Unfrozen: "..target.Name)
    else
        notify("Não estava frozen")
    end
end
local function bring(target)
    if not target or not target.Character or not LocalPlayer.Character then notify("Personagem inválido") return end
    if target.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        target.Character.HumanoidRootPart.CFrame = LocalPlayer.Character.HumanoidRootPart.CFrame * CFrame.new(2,0,0)
        notify("Bring: "..target.Name)
    end
end
local function teleportTo(target)
    if not target or not target.Character or not LocalPlayer.Character then notify("Personagem inválido") return end
    if target.Character:FindFirstChild("HumanoidRootPart") and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
        LocalPlayer.Character.HumanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame + Vector3.new(2,0,0)
        notify("Teleportado para: "..target.Name)
    end
end
local function killPlayer(target)
    if not target or not target.Character then notify("Personagem inválido") return end
    local hum = target.Character:FindFirstChildWhichIsA("Humanoid")
    if hum then
        pcall(function() hum.Health = 0 end)
        notify("Killed: "..target.Name)
    end
end
local function spectate(target)
    if not target or not target.Character then notify("Personagem inválido") return end
    local cam = workspace.CurrentCamera
    if not savedCameraSubject then savedCameraSubject = cam.CameraSubject end
    cam.CameraSubject = target.Character.Humanoid or target.Character
    cam.CameraType = Enum.CameraType.Attach
    notify("Spectating: "..target.Name)
end
local function unspectate()
    local cam = workspace.CurrentCamera
    if savedCameraSubject then cam.CameraSubject = savedCameraSubject cam.CameraType = Enum.CameraType.Custom savedCameraSubject = nil else if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildWhichIsA(\"Humanoid\") then cam.CameraSubject = LocalPlayer.Character:FindFirstChildWhichIsA(\"Humanoid\") cam.CameraType = Enum.CameraType.Custom end end
    notify("Unspectate")
end

-- Buttons (Rayfield will handle layout and scroll)
Window:CreateButton({Name = "Ban (visual)", Callback = function() visualBan(getSelectedPlayer()) end})
Window:CreateButton({Name = "Kick (visual)", Callback = function() visualKick(getSelectedPlayer()) end})
Window:CreateButton({Name = "Jail", Callback = function() jail(getSelectedPlayer()) end})
Window:CreateButton({Name = "Unjail", Callback = function() unjail(getSelectedPlayer()) end})
Window:CreateButton({Name = "Freeze", Callback = function() freezePlayer(getSelectedPlayer()) end})
Window:CreateButton({Name = "Unfreeze", Callback = function() unfreezePlayer(getSelectedPlayer()) end})
Window:CreateButton({Name = "Bring (visual)", Callback = function() bring(getSelectedPlayer()) end})
Window:CreateButton({Name = "Teleport to", Callback = function() teleportTo(getSelectedPlayer()) end})
Window:CreateButton({Name = "Kill (no prompt)", Callback = function() killPlayer(getSelectedPlayer()) end})
Window:CreateButton({Name = "Spectate", Callback = function() spectate(getSelectedPlayer()) end})
Window:CreateButton({Name = "Unspectate", Callback = function() unspectate() end})

notify("Love Painel carregado. Minimizar mostra botão 'Show Love Painel ❤️'.")
Rayfield:Toggle(true)

-- Fim
