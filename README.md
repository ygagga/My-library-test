local MyLibrary = {}

function MyLibrary.new(title)
    local self = {}

    local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local LocalPlayer = game:GetService("Players").LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local HttpService = game:GetService("HttpService")

local MyLib = {
    Elements = {},
    ThemeObjects = {},
    Connections = {},
    Flags = {},
    Themes = {
        Default = {
            Main = Color3.fromRGB(25, 25, 25),
            Second = Color3.fromRGB(32, 32, 32),
            Stroke = Color3.fromRGB(60, 60, 60),
            Divider = Color3.fromRGB(60, 60, 60),
            Text = Color3.fromRGB(240, 240, 240),
            TextDark = Color3.fromRGB(150, 150, 150)
        }
    },
    SelectedTheme = "Default",
    Folder = nil,
    SaveCfg = false
}

local Icons = {}

-- Adicione lógica para carregar ícones se necessário (como feito na Orion)
local Success, Response = pcall(function()
    Icons = HttpService:JSONDecode(game:HttpGetAsync("https://raw.githubusercontent.com/evoincorp/lucideblox/master/src/modules/util/icons.json")).icons
end)

if not Success then
    warn("\nMyLib - Failed to load icons. Error code: " .. Response .. "\n")
end

-- Definir GUI
local MyGui = Instance.new("ScreenGui")
MyGui.Name = "MyLib"
if syn then
    syn.protect_gui(MyGui)
    MyGui.Parent = game.CoreGui
else
    MyGui.Parent = gethui() or game.CoreGui
end

-- Função para verificar se a interface está ativa
function MyLib:IsRunning()
    if gethui then
        return MyGui.Parent == gethui()
    else
        return MyGui.Parent == game:GetService("CoreGui")
    end
end

-- Função para adicionar conexão de sinais
local function AddConnection(Signal, Function)
    if not MyLib:IsRunning() then
        return
    end
    local SignalConnect = Signal:Connect(Function)
    table.insert(MyLib.Connections, SignalConnect)
    return SignalConnect
end

task.spawn(function()
    while MyLib:IsRunning() do
        wait()
    end

    for _, Connection in next, MyLib.Connections do
        Connection:Disconnect()
    end
end)

-- Função para manipular arraste de elementos (semelhante à Orion)
local function AddDraggingFunctionality(DragPoint, Main)
    pcall(function()
        local Dragging, DragInput, MousePos, FramePos = false
        DragPoint.InputBegan:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseButton1 then
                Dragging = true
                MousePos = Input.Position
                FramePos = Main.Position

                Input.Changed:Connect(function()
                    if Input.UserInputState == Enum.UserInputState.End then
                        Dragging = false
                    end
                end)
            end
        end)
        DragPoint.InputChanged:Connect(function(Input)
            if Input.UserInputType == Enum.UserInputType.MouseMovement then
                DragInput = Input
            end
        end)
        UserInputService.InputChanged:Connect(function(Input)
            if Input == DragInput and Dragging then
                local Delta = Input.Position - MousePos
                TweenService:Create(Main, TweenInfo.new(0.45, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {Position  = UDim2.new(FramePos.X.Scale, FramePos.X.Offset + Delta.X, FramePos.Y.Scale, FramePos.Y.Offset + Delta.Y)}):Play()
            end
        end)
    end)
end   

-- Função de criação de elementos
local function Create(Name, Properties, Children)
    local Object = Instance.new(Name)
    for i, v in next, Properties or {} do
        Object[i] = v
    end
    for i, v in next, Children or {} do
        v.Parent = Object
    end
    return Object
end

-- Função para definir propriedades e filhos dos elementos
local function SetProps(Element, Props)
    table.foreach(Props, function(Property, Value)
        Element[Property] = Value
    end)
    return Element
end

local function SetChildren(Element, Children)
    table.foreach(Children, function(_, Child)
        Child.Parent = Element
    end)
    return Element
end

-- Exemplo de criação de um elemento personalizado
MyLib.Elements["Frame"] = function()
    local Frame = Create("Frame", {
        BackgroundColor3 = MyLib.Themes[MyLib.SelectedTheme].Main,
        BorderSizePixel = 0
    })
    return Frame
end

-- Criar interface básica
local MainFrame = MyLib.Elements["Frame"]()
MainFrame.Parent = MyGui
MainFrame.Size = UDim2.new(0, 400, 0, 300)
MainFrame.Position = UDim2.new(0.5, -200, 0.5, -150)

-- Função para adicionar notificações
function MyLib:MakeNotification(NotificationConfig)
    spawn(function()
        NotificationConfig.Name = NotificationConfig.Name or "Notification"
        NotificationConfig.Content = NotificationConfig.Content or "Test"
        NotificationConfig.Time = NotificationConfig.Time or 15

        local NotificationFrame = Create("Frame", {
            BackgroundColor3 = Color3.fromRGB(25, 25, 25),
            Size = UDim2.new(1, 0, 0, 50),
            Position = UDim2.new(0.5, -150, 0, 0)
        })

        local Title = Create("TextLabel", {
            Text = NotificationConfig.Name,
            TextSize = 18,
            TextColor3 = Color3.fromRGB(240, 240, 240),
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 10, 0, 5)
        })
        Title.Parent = NotificationFrame

        local Content = Create("TextLabel", {
            Text = NotificationConfig.Content,
            TextSize = 14,
            TextColor3 = Color3.fromRGB(200, 200, 200),
            BackgroundTransparency = 1,
            Position = UDim2.new(0, 10, 0, 30)
        })
        Content.Parent = NotificationFrame

        NotificationFrame.Parent = MyGui

        -- Animar a notificação
        TweenService:Create(NotificationFrame, TweenInfo.new(0.5, Enum.EasingStyle.Quint), {Position = UDim2.new(0.5, -150, 0.1, 0)}):Play()

        wait(NotificationConfig.Time)
        NotificationFrame:Destroy()
    end)
end

-- Função de inicialização
function MyLib:Init()
    -- Lógica de inicialização
end

-- Criar uma janela exemplo
local Window = MyLib:MakeWindow({
    Name = "My Custom Library",
    SaveConfig = true
})

    
    self.Tabs = {}

    -- Função para adicionar abas
    function self:Tab(name)
        local Tab = {}

        local TabButton = Instance.new("TextButton")
        TabButton.Parent = MainFrame
        TabButton.Size = UDim2.new(0, 80, 0, 25)
        TabButton.Position = UDim2.new(0, #self.Tabs * 85, 0, 35)
        TabButton.Text = name
        TabButton.BackgroundColor3 = Color3.fromRGB(45, 45, 45)
        TabButton.TextColor3 = Color3.fromRGB(255, 255, 255)
        TabButton.Font = Enum.Font.Gotham
        TabButton.TextSize = 14

        local TabFrame = Instance.new("Frame")
        TabFrame.Parent = MainFrame
        TabFrame.Size = UDim2.new(1, 0, 1, -60)
        TabFrame.Position = UDim2.new(0, 0, 0, 60)
        TabFrame.BackgroundTransparency = 1
        TabFrame.Visible = #self.Tabs == 0

        function Tab:Button(text, callback)
            local Button = Instance.new("TextButton")
            Button.Parent = TabFrame
            Button.Size = UDim2.new(0.9, 0, 0, 30)
            Button.Position = UDim2.new(0.05, 0, 0, #TabFrame:GetChildren() * 35)
            Button.Text = text
            Button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
            Button.TextColor3 = Color3.fromRGB(255, 255, 255)
            Button.Font = Enum.Font.Gotham
            Button.TextSize = 14
            Button.MouseButton1Click:Connect(callback)
        end

        function Tab:Toggle(text, callback)
            local Toggle = Instance.new("TextButton")
            Toggle.Parent = TabFrame
            Toggle.Size = UDim2.new(0.9, 0, 0, 30)
            Toggle.Position = UDim2.new(0.05, 0, 0, #TabFrame:GetChildren() * 35)
            Toggle.Text = text .. " [OFF]"
            Toggle.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
            Toggle.TextColor3 = Color3.fromRGB(255, 255, 255)
            Toggle.Font = Enum.Font.Gotham
            Toggle.TextSize = 14

            local state = false
            Toggle.MouseButton1Click:Connect(function()
                state = not state
                Toggle.Text = text .. (state and " [ON]" or " [OFF]")
                callback(state)
            end)
        end

        -- Alternar abas
        TabButton.MouseButton1Click:Connect(function()
            for _, v in pairs(MainFrame:GetChildren()) do
                if v:IsA("Frame") and v ~= MainFrame then
                    v.Visible = false
                end
            end
            TabFrame.Visible = true
        end)

        table.insert(self.Tabs, Tab)
        return Tab
    end

    return self
end

return MyLibrary
