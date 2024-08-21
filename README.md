local player = game.Players.LocalPlayer
local screenGui = Instance.new("ScreenGui", player:WaitForChild("PlayerGui"))

-- Função para tornar os botões móveis
local function makeButtonMovable(button)
    local dragging = false
    local dragInput, mousePos, framePos

    button.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            mousePos = input.Position
            framePos = button.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End then
                    dragging = false
                end
            end)
        end
    end)

    button.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
            dragInput = input
        end
    end)

    game:GetService("UserInputService").InputChanged:Connect(function(input)
        if input == dragInput and dragging then
            local delta = input.Position - mousePos
            button.Position = UDim2.new(framePos.X.Scale, framePos.X.Offset + delta.X, framePos.Y.Scale, framePos.Y.Offset + delta.Y)
        end
    end)
end

-- Configuração do botão de ativar/desativar velocidade
local speedButton = Instance.new("TextButton", screenGui)
speedButton.Size = UDim2.new(0, 200, 0, 50)
speedButton.Position = UDim2.new(0.5, -100, 0.8, 0)
speedButton.Text = "Ativar Speed"
makeButtonMovable(speedButton)

-- Configuração dos botões de aumentar e diminuir velocidade
local increaseButton = Instance.new("TextButton", screenGui)
increaseButton.Size = UDim2.new(0, 50, 0, 50)
increaseButton.Position = UDim2.new(0.5, 110, 0.8, 0)
increaseButton.Text = "+"
makeButtonMovable(increaseButton)

local decreaseButton = Instance.new("TextButton", screenGui)
decreaseButton.Size = UDim2.new(0, 50, 0, 50)
decreaseButton.Position = UDim2.new(0.5, -160, 0.8, 0)
decreaseButton.Text = "-"
makeButtonMovable(decreaseButton)

local speedLabel = Instance.new("TextLabel", screenGui)
speedLabel.Size = UDim2.new(0, 100, 0, 50)
speedLabel.Position = UDim2.new(0.5, -50, 0.7, 0)
speedLabel.Text = "Vel: 2"
speedLabel.TextScaled = true
makeButtonMovable(speedLabel)

-- Configuração do botão de minimizar/maximizar GUI
local minimizeButton = Instance.new("TextButton", screenGui)
minimizeButton.Size = UDim2.new(0, 50, 0, 50)
minimizeButton.Position = UDim2.new(0.5, -25, 0.6, 0)
minimizeButton.Text = "-"
makeButtonMovable(minimizeButton)

-- Variáveis para a velocidade e controle do script
local speedActive = false
local distancePerTP = 2 -- Valor inicial da velocidade
local interval = 0.05 -- Intervalo entre os teletransportes (em segundos)
local guiMinimized = false

local function startSpeed()
    local character = player.Character or player.CharacterAdded:Wait()
    local rootPart = character:FindFirstChild("RootPart") or character:FindFirstChild("HumanoidRootPart")
    
    if not rootPart then
        warn("RootPart não encontrado!")
        return
    end

    speedActive = true
    while speedActive do
        -- Verifica se a criatura está se movendo
        if rootPart.Velocity.Magnitude > 0 then
            -- Calcula a nova posição da criatura com base na direção do movimento
            local moveDirection = rootPart.CFrame.LookVector
            local newPosition = rootPart.Position + (moveDirection * distancePerTP)
            -- Define a nova posição
            rootPart.CFrame = CFrame.new(newPosition, newPosition + moveDirection)
        end
        -- Espera o próximo TP
        wait(interval)
    end
end

local function stopSpeed()
    speedActive = false
end

-- Função para aumentar a velocidade
increaseButton.MouseButton1Click:Connect(function()
    distancePerTP = math.min(distancePerTP + 1, 30) -- Limite máximo de 30
    speedLabel.Text = "Vel: " .. distancePerTP
end)

-- Função para diminuir a velocidade
decreaseButton.MouseButton1Click:Connect(function()
    distancePerTP = math.max(distancePerTP - 1, 1) -- Limite mínimo de 1
    speedLabel.Text = "Vel: " .. distancePerTP
end)

-- Função para ativar/desativar a velocidade
speedButton.MouseButton1Click:Connect(function()
    if speedActive then
        stopSpeed()
        speedButton.Text = "Ativar Speed"
    else
        startSpeed()
        speedButton.Text = "Desativar Speed"
    end
end)

-- Função para minimizar e maximizar a GUI, incluindo suporte para toque
minimizeButton.MouseButton1Click:Connect(function()
    guiMinimized = not guiMinimized

    if guiMinimized then
        speedButton.Visible = false
        increaseButton.Visible = false
        decreaseButton.Visible = false
        speedLabel.Visible = false
        minimizeButton.Text = "+"
    else
        speedButton.Visible = true
        increaseButton.Visible = true
        decreaseButton.Visible = true
        speedLabel.Visible = true
        minimizeButton.Text = "-"
    end
end)
