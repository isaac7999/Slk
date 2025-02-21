-- Criando o painel principal
local ScreenGui = Instance.new("ScreenGui")
local MainFrame = Instance.new("Frame")
local MinimizeButton = Instance.new("TextButton")
local TitleLabel = Instance.new("TextLabel")

-- Configuração do painel principal
ScreenGui.Parent = game.Players.LocalPlayer:WaitForChild("PlayerGui")
MainFrame.Size = UDim2.new(0, 300, 0, 450)
MainFrame.Position = UDim2.new(0.5, -150, 0.5, -225)
MainFrame.BackgroundColor3 = Color3.fromRGB(173, 216, 230)
MainFrame.Parent = ScreenGui

-- Título do painel
TitleLabel.Size = UDim2.new(1, 0, 0, 30)
TitleLabel.Position = UDim2.new(0, 0, 0, 0)
TitleLabel.BackgroundTransparency = 1
TitleLabel.Text = "SCRIPT PAGO SONICX"
TitleLabel.TextSize = 20
TitleLabel.TextColor3 = Color3.fromRGB(0, 0, 0)
TitleLabel.Parent = MainFrame

-- Botão de minimizar
MinimizeButton.Size = UDim2.new(0, 50, 0, 25)
MinimizeButton.Position = UDim2.new(1, -55, 0, 5)
MinimizeButton.BackgroundColor3 = Color3.fromRGB(255, 0, 0)
MinimizeButton.Text = "_"
MinimizeButton.Parent = MainFrame
MinimizeButton.MouseButton1Click:Connect(function()
    MainFrame.Visible = not MainFrame.Visible
end)

-- Função Anti-Detectado e Anti-Ban
local function antiDetect()
    local mt = getrawmetatable(game)
    setreadonly(mt, false)
    local oldIndex = mt.__index
    mt.__index = newcclosure(function(self, key)
        if key == "Kick" or key == "Destroy" then
            return function() end
        end
        return oldIndex(self, key)
    end)
end
antiDetect()

-- Função para alternar cor do botão
local function toggleButton(button, state)
    if state then
        button.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Verde
    else
        button.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho
    end
end

-- Criando os botões e funcionalidades
local function createButton(name, pos, callback)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 100, 0, 25)
    button.Position = UDim2.new(0, 10, 0, pos)
    button.BackgroundColor3 = Color3.fromRGB(255, 0, 0) -- Vermelho por padrão
    button.Text = name
    button.Parent = MainFrame
    local state = false
    button.MouseButton1Click:Connect(function()
        state = not state
        toggleButton(button, state)
        callback(state)
    end)
    return button
end

-- Função Aimbot
local function aimbot(state)
    local player = game.Players.LocalPlayer
    local camera = game.Workspace.CurrentCamera
    local userInputService = game:GetService("UserInputService")

    if state then
        local function aimAtTarget()
            while state do
                wait()
                local closestPlayer = nil
                local closestMagnitude = math.huge

                -- Encontrar o jogador mais próximo
                for _, targetPlayer in pairs(game.Players:GetPlayers()) do
                    if targetPlayer ~= player and targetPlayer.Character and targetPlayer.Character:FindFirstChild("Head") then
                        local targetPos = targetPlayer.Character.Head.Position
                        local screenPos, onScreen = camera:WorldToScreenPoint(targetPos)
                        local magnitude = (Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(userInputService:GetMouseLocation().X, userInputService:GetMouseLocation().Y)).Magnitude

                        if onScreen and magnitude < closestMagnitude then
                            closestMagnitude = magnitude
                            closestPlayer = targetPlayer
                        end
                    end
                end

                -- Se encontrar um alvo, mover a mira para a cabeça do jogador
                if closestPlayer then
                    local targetPos = closestPlayer.Character.Head.Position
                    local screenPos = camera:WorldToScreenPoint(targetPos)
                    local aimDirection = Vector2.new(screenPos.X, screenPos.Y) - Vector2.new(userInputService:GetMouseLocation().X, userInputService:GetMouseLocation().Y)

                    -- Atualiza a direção do toque (para mobile)
                    userInputService.InputChanged:Connect(function(input)
                        if input.UserInputType == Enum.UserInputType.Touch then
                            -- Controla a mira conforme o toque do jogador
                            userInputService.MouseBehavior = Enum.MouseBehavior.LockCenter
                        end
                    end)
                end
            end
        end
        aimAtTarget()
    else
        -- Desliga o aimbot quando ele é desativado
        userInputService.MouseBehavior = Enum.MouseBehavior.Default
    end
end
createButton("Aimbot", 40, aimbot)

-- ESP Player
local function espPlayer(state)
    for _, player in pairs(game.Players:GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            if state then
                local highlight = Instance.new("Highlight", player.Character)
                highlight.FillColor = Color3.fromRGB(255, 0, 0)
                highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                highlight.FillTransparency = 0.5
            else
                if player.Character:FindFirstChild("Highlight") then
                    player.Character.Highlight:Destroy()
                end
            end
        end
    end
end
createButton("ESP Player", 70, espPlayer)

-- Caixa 2D (Hitbox visual)
local function caixa2D(state)
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            if state then
                local box = Instance.new("BoxHandleAdornment")
                box.Size = Vector3.new(4, 6, 4)
                box.Adornee = player.Character.HumanoidRootPart
                box.Color3 = Color3.fromRGB(0, 255, 0)
                box.AlwaysOnTop = true
                box.ZIndex = 10
                box.Parent = player.Character
            else
                for _, obj in pairs(player.Character:GetChildren()) do
                    if obj:IsA("BoxHandleAdornment") then
                        obj:Destroy()
                    end
                end
            end
        end
    end
end
createButton("Caixa 2D", 100, caixa2D)

-- Função de "Bring Car" (Trazer Carro)
local function bringCar(state)
    if state then
        local car = game.Workspace:FindFirstChild("Car") -- Verifica se há um carro no jogo
        if car then
            local player = game.Players.LocalPlayer
            if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                car:SetPrimaryPartCFrame(player.Character.HumanoidRootPart.CFrame)
            end
        end
    end
end
createButton("Bring Car", 130, bringCar)

-- Auto CL
local function autoCL(state)
    if state then
        game:GetService("RunService").Stepped:Connect(function()
            if game.Players.LocalPlayer.Character and game.Players.LocalPlayer.Character:FindFirstChild("Humanoid") then
                if game.Players.LocalPlayer.Character.Humanoid.Health == 0 then
                    game.Players.LocalPlayer:Kick("Removido pelo servidor.")
                end
            end
        end)
    end
end
createButton("Auto CL", 160, autoCL)

-- ESP Line
local function espLine(state)
    local player = game.Players.LocalPlayer
    local playerChar = player.Character
    local lineBeams = {} -- Para armazenar as linhas para cada jogador
    
    if state then
        -- Atualizando a linha constantemente se o jogador estiver perto
        game:GetService("RunService").Heartbeat:Connect(function()
            for _, targetPlayer in pairs(game.Players:GetPlayers()) do
                if targetPlayer ~= player then
                    local targetChar = targetPlayer.Character
                    if targetChar and targetChar:FindFirstChild("HumanoidRootPart") then
                        local distance = (playerChar.HumanoidRootPart.Position - targetChar.HumanoidRootPart.Position).Magnitude
                        -- Só cria a linha se o jogador estiver perto
                        if distance < 50 then
                            -- Criando o Beam entre as cabeças
                            if not lineBeams[targetPlayer] then
                                local attachment1 = Instance.new("Attachment", playerChar.HumanoidRootPart)
                                local attachment2 = Instance.new("Attachment", targetChar.HumanoidRootPart)
                                local beam = Instance.new("Beam")
                                beam.Attachment0 = attachment1
                                beam.Attachment1 = attachment2
                                beam.Parent = playerChar
                                beam.Color = ColorSequence.new(Color3.fromRGB(255, 0, 0))
                                lineBeams[targetPlayer] = beam
                            end
                        else
                            -- Se estiver longe, destrói a linha
                            if lineBeams[targetPlayer] then
                                lineBeams[targetPlayer]:Destroy()
                                lineBeams[targetPlayer] = nil
                            end
                        end
                    end
                end
            end
        end)
    else
        -- Destruir todas as linhas se desativado
        for _, beam in pairs(lineBeams) do
            beam:Destroy()
        end
        lineBeams = {} -- Limpar o armazenamento
    end
end
createButton("ESP Line", 190, espLine)

-- ESP Nick
local function espNick(state)
    if state then
        for _, player in pairs(game.Players:GetPlayers()) do
            if player.Character and not player.Character:FindFirstChild("BillboardGui") then
                local gui = Instance.new("BillboardGui", player.Character)
                gui.Size = UDim2.new(0, 100, 0, 50)
                gui.AlwaysOnTop = true
                local text = Instance.new("TextLabel", gui)
                text.Size = UDim2.new(1, 0, 1, 0)
                text.Text = player.Name
                text.TextColor3 = Color3.fromRGB(0, 0, 0) -- Nome Preto
                text.TextSize = 14 -- Nome Menor
                text.BackgroundTransparency = 1
            end
        end
    else
        for _, player in pairs(game.Players:GetPlayers()) do
            if player.Character and player.Character:FindFirstChild("BillboardGui") then
                player.Character.BillboardGui:Destroy()
            end
        end
    end
end
createButton("ESP Nick", 220, espNick)

-- Hitbox
local function hitbox(state)
    for _, player in pairs(game.Players:GetPlayers()) do
        if player.Character and player.Character:FindFirstChild("Head") then
            player.Character.Head.Size = state and Vector3.new(5, 5, 5) or Vector3.new(1, 1, 1)
        end
    end
end
createButton("Hitbox", 250, hitbox)

-- Teleporte para diferentes locais
local function createTeleportButton(name, pos, targetPosition)
    local button = Instance.new("TextButton")
    button.Size = UDim2.new(0, 150, 0, 30)
    button.Position = UDim2.new(0, 75, 0, pos)
    button.BackgroundColor3 = Color3.fromRGB(0, 255, 0) -- Verde
    button.Text = name
    button.Parent = MainFrame
    button.MouseButton1Click:Connect(function()
        local player = game.Players.LocalPlayer
        if player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
            player.Character.HumanoidRootPart.CFrame = targetPosition
        end
    end)
    return button
end

-- Botões de teleporte
createTeleportButton("Teleporte Lavagem", 250, CFrame.new(19764.6, 69.6, 13152.5))
createTeleportButton("Teleporte Ilegal", 290, CFrame.new(12016.9, 30.0, 12810.3))
createTeleportButton("Teleporte Praça", 330, CFrame.new(-297.6, 4.8, 384.2))
createTeleportButton("Teleporte Lojinha", 370, CFrame.new(-71.9, 6.0, -111.9))
