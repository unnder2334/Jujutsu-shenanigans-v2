--// DebreeRef Visual Deleter (Ultra Performance) - By light_use182
local workspace = game:GetService("Workspace")

--// 1. ASSINATURA AO EXECUTAR
local function showCredit()
    local screenGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
    local msg = Instance.new("TextLabel", screenGui)
    msg.Size = UDim2.new(0, 400, 0, 50)
    msg.Position = UDim2.new(0.5, -200, 0.4, 0)
    msg.BackgroundTransparency = 1
    msg.Text = "By light_use182"
    msg.TextColor3 = Color3.new(1, 1, 1)
    msg.Font = Enum.Font.GothamBold
    msg.TextSize = 25
    task.delay(2, function() screenGui:Destroy() end)
end
showCredit()

--// 2. FUNÇÃO PARA DELETAR O ESTILHAÇO
local function handleObject(obj)
    -- Verifica se o nome é exatamente o que o senhor pediu
    if obj.Name == "DebreeRef" then
        obj:Destroy() -- Deleta instantaneamente
    end
end

--// 3. LIMPEZA INICIAL (Caso já existam estilhaços no mapa)
for _, item in pairs(workspace:GetDescendants()) do
    handleObject(item)
end

--// 4. MONITORAMENTO EM TEMPO REAL (SEM LOOP / SEM LAG)
-- O 'DescendantAdded' é um evento: ele não gasta processamento enquanto nada acontece.
-- Ele só "acorda" quando algo novo é criado no mapa.
workspace.DescendantAdded:Connect(function(newObj)
    if newObj.Name == "DebreeRef" then
        newObj:Destroy()
    end
end)

print("Monitoramento de DebreeRef ativado. By light_use182")
--// Ultra Light FPS Booster (No Lag) - By light_use182
local workspace = game:GetService("Workspace")

--// 1. CONFIGURAÇÕES
local materialFixo = Enum.Material.SmoothPlastic

--// 2. FUNÇÃO DE LIMPEZA (Só roda quando necessário)
local function otimizar(obj)
    if obj:IsA("BasePart") then
        -- Torna o material leve
        obj.Material = materialFixo
        
        -- Se for estilhaço (Debris/Fragment), deixa invisível para ganhar FPS
        local nome = obj.Name:lower()
        if nome:find("debris") or nome:find("frag") or nome:find("piece") then
            obj.Transparency = 1
            obj.CanCollide = false -- Evita cálculos de física desnecessários
        end

        -- Remove texturas e adesivos internos
        for _, child in pairs(obj:GetChildren()) do
            if child:IsA("Texture") or child:IsA("Decal") then
                child:Destroy()
            end
        end
    elseif obj:IsA("MeshPart") then
        obj.TextureID = ""
        obj.Material = materialFixo
    end
end

--// 3. VARREDURA INICIAL (Roda apenas UMA VEZ ao ligar)
for _, item in pairs(workspace:GetDescendants()) do
    otimizar(item)
end

--// 4. O SEGREDO DO FPS: Monitorar apenas o que entra no jogo
-- Em vez de um loop infinito, o script "dorme" e só acorda quando um bloco quebra ou volta
workspace.DescendantAdded:Connect(function(item)
    -- Espera um frame para o objeto carregar as propriedades
    task.wait() 
    if item:IsA("BasePart") or item:IsA("MeshPart") then
        otimizar(item)
    end
end)

print("FPS Boost Ativado (Versão Leve). By light_use182")
--// Cam + Boneco Lock 360 (Players + NPCs) - By light_use182
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")
local CoreGui = game:GetService("CoreGui")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

local camEnabled = false
local lockedTarget = nil 

--// 1. CRÉDITOS
local function showCredit()
    local screenGui = Instance.new("ScreenGui", CoreGui)
    screenGui.Name = "AimCredit"
    local msg = Instance.new("TextLabel", screenGui)
    msg.Size = UDim2.new(0, 400, 0, 50)
    msg.Position = UDim2.new(0.5, -200, 0.4, 0)
    msg.BackgroundTransparency = 1
    msg.Text = "By light_use182"
    msg.TextColor3 = Color3.new(1, 1, 1)
    msg.Font = Enum.Font.GothamBold
    msg.TextSize = 25
    task.delay(2, function() screenGui:Destroy() end)
end
showCredit()

--// 2. FUNÇÃO DE ARRASTAR
local function makeDraggable(gui)
    local dragging, dragInput, dragStart, startPos
    gui.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true; dragStart = input.Position; startPos = gui.Position
            input.Changed:Connect(function() if input.UserInputState == Enum.UserInputState.End then dragging = false end end)
        end
    end)
    gui.InputChanged:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then dragInput = input end
    end)
    UIS.InputChanged:Connect(function(input)
        if dragging and input == dragInput then
            local delta = input.Position - dragStart
            gui.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

--// 3. BUSCA DE ALVO (PLAYERS + NPCs) NO FOCO DA TELA
local function getTarget()
    local closestToCenter = nil
    local shortestDistance = math.huge
    local centerScreen = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)

    -- Varre o Workspace inteiro atrás de modelos com Humanoid
    for _, obj in pairs(workspace:GetDescendants()) do
        if obj:IsA("Humanoid") and obj.Parent:IsA("Model") then
            local char = obj.Parent
            local hrp = char:FindFirstChild("HumanoidRootPart")
            
            -- Não focar em si mesmo e verificar se está vivo
            if char ~= LocalPlayer.Character and hrp and obj.Health > 0 then
                local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                
                if onScreen then
                    local distToCenter = (Vector2.new(screenPos.X, screenPos.Y) - centerScreen).Magnitude
                    if distToCenter < shortestDistance then
                        shortestDistance = distToCenter
                        closestToCenter = hrp
                    end
                end
            end
        end
    end
    return closestToCenter
end

--// 4. UI DO BOTÃO
local ScreenGui = Instance.new("ScreenGui", CoreGui)
local aimBtn = Instance.new("TextButton", ScreenGui)
aimBtn.Size = UDim2.new(0, 50, 0, 50)
aimBtn.Position = UDim2.new(0.5, -25, 0.1, 0)
aimBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
aimBtn.Text = "aim"
aimBtn.TextColor3 = Color3.new(1, 1, 1)
aimBtn.Font = Enum.Font.GothamBold
Instance.new("UICorner", aimBtn).CornerRadius = UDim.new(1, 0)
makeDraggable(aimBtn)

--// 5. ATIVAÇÃO
aimBtn.MouseButton1Click:Connect(function()
    camEnabled = not camEnabled
    if camEnabled then
        lockedTarget = getTarget()
        if lockedTarget then
            aimBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 255)
            Camera.CameraType = Enum.CameraType.Scriptable
        else
            camEnabled = false
        end
    else
        lockedTarget = nil
        aimBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
        Camera.CameraType = Enum.CameraType.Custom
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
            LocalPlayer.Character:FindFirstChildOfClass("Humanoid").AutoRotate = true
        end
    end
end)

--// 6. LOOP DE MANUTENÇÃO
RunService.RenderStepped:Connect(function()
    if camEnabled and lockedTarget and LocalPlayer.Character then
        local myHRP = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
        local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
        
        -- Verifica se o alvo (Player ou NPC) ainda existe e está vivo
        if not lockedTarget.Parent or not lockedTarget.Parent:FindFirstChildOfClass("Humanoid") or lockedTarget.Parent:FindFirstChildOfClass("Humanoid").Health <= 0 then
            camEnabled = false
            lockedTarget = nil
            aimBtn.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
            Camera.CameraType = Enum.CameraType.Custom
            return
        end

        if myHRP and humanoid then
            humanoid.AutoRotate = false
            local targetPos = lockedTarget.Position
            
            -- Gira o boneco
            local lookAtBody = Vector3.new(targetPos.X, myHRP.Position.Y, targetPos.Z)
            myHRP.CFrame = CFrame.lookAt(myHRP.Position, lookAtBody)

            -- Gira a câmera
            local camPos = myHRP.Position + (myHRP.CFrame.LookVector * -10) + Vector3.new(0, 5, 0)
            Camera.CFrame = CFrame.lookAt(camPos, targetPos)
        end
    end
end)
