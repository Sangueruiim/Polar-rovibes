local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local LocalPlayer = Players.LocalPlayer

-- VERIFICAÇÃO DE NICK
local NICK_PERMITIDO = "SteikeX689"
if LocalPlayer.Name ~= NICK_PERMITIDO then
    -- Mostra aviso e para o script
    local playerGui = LocalPlayer:WaitForChild("PlayerGui")
    local warningGui = Instance.new("ScreenGui")
    local warningLabel = Instance.new("TextLabel")
    
    warningGui.Name = "WarningGui"
    warningGui.ResetOnSpawn = false
    warningGui.Parent = playerGui
    
    warningLabel.Size = UDim2.new(0, 400, 0, 100)
    warningLabel.Position = UDim2.new(0.5, -200, 0.5, -50)
    warningLabel.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
    warningLabel.BackgroundTransparency = 0.3
    warningLabel.TextColor3 = Color3.fromRGB(255, 50, 50)
    warningLabel.Font = Enum.Font.GothamBold
    warningLabel.TextSize = 20
    warningLabel.Text = "❌ VOCÊ NÃO TEM PERMISSÃO!\nApenas SteikeX689 pode usar este script."
    warningLabel.TextWrapped = true
    warningLabel.Parent = warningGui
    
    warn("Script bloqueado: Apenas SteikeX689 pode usar este script.")
    return -- Para o script completamente
end

-- Localiza o sistema de baús baseado no script original
local Remotes = ReplicatedStorage:WaitForChild("Remotes", 5)
local ChestSpawn = Remotes and Remotes:WaitForChild("ChestSpawn", 5)
local GetChestTarget = ChestSpawn and ChestSpawn:WaitForChild("GetChestTarget", 5)

if not GetChestTarget then
    warn("Não foi possível encontrar a Remote do Baú. Certifique-se de estar no jogo correto.")
    return
end

-- Criando a interface (Botão na Tela)
local ScreenGui = Instance.new("ScreenGui")
local MainButton = Instance.new("TextButton")
local UICorner = Instance.new("UICorner")

ScreenGui.Name = "ChestTpGui"
ScreenGui.ResetOnSpawn = false
-- Tenta colocar no CoreGui para não sumir, senão vai para PlayerGui
local success, _ = pcall(function() ScreenGui.Parent = game:GetService("CoreGui") end)
if not success then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

-- Estilização do Botão
MainButton.Name = "TpButton"
MainButton.Size = UDim2.new(0, 180, 0, 45)
MainButton.Position = UDim2.new(0.5, -90, 0.15, 0) -- Centralizado no topo da tela
MainButton.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
MainButton.TextColor3 = Color3.fromRGB(255, 255, 255)
MainButton.Font = Enum.Font.GothamBold
MainButton.TextSize = 16
MainButton.Text = "Buscar e Teleportar"
MainButton.Active = true
MainButton.Draggable = true -- Você pode arrastar o botão para onde quiser na tela
MainButton.Parent = ScreenGui

UICorner.CornerRadius = UDim.new(0, 8)
UICorner.Parent = MainButton

-- Lógica do Teleporte
MainButton.MouseButton1Click:Connect(function()
    MainButton.Text = "Buscando baú..."
    MainButton.BackgroundColor3 = Color3.fromRGB(200, 150, 50)

    -- Invoca o servidor para pegar a tabela de dados do baú  
    local success, result = pcall(function()  
        return GetChestTarget:InvokeServer()  
    end)  

    if success and type(result) == "table" and result.active == true and result.pos then  
        local targetPos = result.pos  
        local character = LocalPlayer.Character  
        local rootPart = character and character:FindFirstChild("HumanoidRootPart")  

        if rootPart then  
            -- Teleporta 3 studs acima do baú para evitar prender no chão  
            rootPart.CFrame = CFrame.new(targetPos + Vector3.new(0, 3, 0))  

            MainButton.Text = "Teleportado!"  
            MainButton.BackgroundColor3 = Color3.fromRGB(50, 150, 50)  
        else  
            MainButton.Text = "Erro: Sem Corpo"  
            MainButton.BackgroundColor3 = Color3.fromRGB(150, 50, 50)  
        end  
    else  
        MainButton.Text = "Nenhum baú ativo"  
        MainButton.BackgroundColor3 = Color3.fromRGB(150, 50, 50)  
    end  

    -- Reseta o botão após 1.5 segundos  
    task.wait(1.5)  
    MainButton.Text = "Buscar e Teleportar"  
    MainButton.BackgroundColor3 = Color3.fromRGB(35, 35, 45)
end)
