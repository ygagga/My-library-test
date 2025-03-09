local MyLibrary = {}

function MyLibrary.new(title)
    local self = {}

    -- Criar ScreenGui
    local ScreenGui = Instance.new("ScreenGui")
    ScreenGui.Parent = game.CoreGui
    ScreenGui.Name = title

    -- Criar Main Frame
    local MainFrame = Instance.new("Frame")
    MainFrame.Parent = ScreenGui
    MainFrame.Size = UDim2.new(0, 400, 0, 300)
    MainFrame.Position = UDim2.new(0.5, -200, 0.5, -150)
    MainFrame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)

    -- Criar UICorner
    local UICorner = Instance.new("UICorner")
    UICorner.CornerRadius = UDim.new(0, 10)
    UICorner.Parent = MainFrame

    -- Criar título
    local Title = Instance.new("TextLabel")
    Title.Parent = MainFrame
    Title.Size = UDim2.new(1, 0, 0, 30)
    Title.Text = title
    Title.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    Title.TextColor3 = Color3.fromRGB(255, 255, 255)
    Title.Font = Enum.Font.GothamBold
    Title.TextSize = 16

    self.MainFrame = MainFrame
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
