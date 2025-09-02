-- Vitin Hub – KNRL Edition Fly Mobile
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera

-- Controles
local ESPPlayersOn, ESPBrainOn = false,false
local SpeedOn, JumpOn, FlyOn, NoclipOn = false,false,false,false
local AimbotOn = false
local PlayerHighlights, BrainHighlights = {},{}
local FlyVelocity = Vector3.new()

-- GUI simplificada
local ScreenGui = Instance.new("ScreenGui", game.CoreGui)
local Frame = Instance.new("Frame", ScreenGui)
Frame.Size = UDim2.new(0,180,0,220)
Frame.Position = UDim2.new(0.35,0,0.3,0)
Frame.BackgroundColor3 = Color3.fromRGB(25,25,25)
Frame.Active = true
Frame.Draggable = true

local MinBtn = Instance.new("TextButton", Frame)
MinBtn.Size = UDim2.new(0,20,0,20)
MinBtn.Position = UDim2.new(1,-25,0,5)
MinBtn.Text = "-"
local CloseBtn = Instance.new("TextButton", Frame)
CloseBtn.Size = UDim2.new(0,20,0,20)
CloseBtn.Position = UDim2.new(1,-50,0,5)
CloseBtn.Text = "X"
local EmojiBtn = Instance.new("TextButton", ScreenGui)
EmojiBtn.Size = UDim2.new(0,30,0,30)
EmojiBtn.Position = UDim2.new(0.1,0,0.8,0)
EmojiBtn.Text="🥷"
EmojiBtn.Visible=false
EmojiBtn.Active=true
EmojiBtn.Draggable=true

MinBtn.MouseButton1Click:Connect(function() Frame.Visible=false EmojiBtn.Visible=true end)
EmojiBtn.MouseButton1Click:Connect(function() Frame.Visible=true EmojiBtn.Visible=false end)
CloseBtn.MouseButton1Click:Connect(function() ScreenGui:Destroy() end)

-- Abas e botões
local Tabs={"Mov","Util"}
local CurrentTab="Mov"
local OptionsFrame = Instance.new("Frame", Frame)
OptionsFrame.Size = UDim2.new(1,-10,1,-65)
OptionsFrame.Position = UDim2.new(0,5,0,60)
OptionsFrame.BackgroundTransparency = 1
local Buttons={}

local function CreateButton(name,callback,row,col)
    local btn = Instance.new("TextButton", OptionsFrame)
    btn.Size = UDim2.new(0,50,0,30)
    btn.Position = UDim2.new(0,(col-1)*60,0,(row-1)*35)
    btn.Text = name
    btn.TextSize = 12
    btn.BackgroundColor3 = Color3.fromRGB(55,55,55)
    btn.TextColor3 = Color3.fromRGB(255,255,255)
    btn.MouseButton1Click:Connect(callback)
    table.insert(Buttons,btn)
end

local function UpdateOptions()
    for _,b in pairs(Buttons) do b:Destroy() end
    Buttons={}
    if CurrentTab=="Mov" then
        CreateButton("⚡ Speed",function() SpeedOn = not SpeedOn end,1,1)
        CreateButton("🦘 Jump",function() JumpOn = not JumpOn end,1,2)
        CreateButton("🛸 Fly",function() FlyOn = not FlyOn end,1,3)
    elseif CurrentTab=="Util" then
        CreateButton("🚧 Noclip",function() NoclipOn = not NoclipOn end,1,1)
        CreateButton("👤 ESP Players",function() ESPPlayersOn = not ESPPlayersOn end,1,2)
        CreateButton("🔥 ESP Brain",function() ESPBrainOn = not ESPBrainOn end,1,3)
        CreateButton("🎯 Aimbot",function() AimbotOn = not AimbotOn end,2,1)
        CreateButton("♻ Reset",function() LocalPlayer.Character:BreakJoints() end,2,2)
        CreateButton("⏩ Hop Server",function() print("Hop Server") end,2,3)
    end
end

for i,t in pairs(Tabs) do
    local btn = Instance.new("TextButton", Frame)
    btn.Size=UDim2.new(0,80,0,25)
    btn.Position=UDim2.new(0,5+(i-1)*85,0,30)
    btn.Text=t
    btn.MouseButton1Click:Connect(function() CurrentTab=t UpdateOptions() end)
end

UpdateOptions()

-- RunService
RunService.RenderStepped:Connect(function()
    local char = LocalPlayer.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end

    -- Super Pulo
    if JumpOn then hum.UseJumpPower=true hum.JumpPower=150 else hum.JumpPower=50 end

    -- Speed
    hum.WalkSpeed = SpeedOn and 80 or 16

    -- Fly analógico
    if FlyOn then
        root.Anchored=false
        local bv=root:FindFirstChild("FlyBV") or Instance.new("BodyVelocity",root)
        bv.Name="FlyBV"
        bv.MaxForce=Vector3.new(1e5,1e5,1e5)
        bv.Velocity=FlyVelocity + (UserInputService:IsKeyDown(Enum.KeyCode.Space) and Vector3.new(0,30,0) or Vector3.new())
        bv.Parent=root
    else
        local bv=root:FindFirstChild("FlyBV")
        if bv then bv:Destroy() end
    end

    -- Noclip
    if NoclipOn then
        for _,p in pairs(char:GetDescendants()) do
            if p:IsA("BasePart") then p.CanCollide=false end
        end
    end

    -- ESP Players
    if ESPPlayersOn then
        for _,plr in pairs(Players:GetPlayers()) do
            if plr~=LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
                if not PlayerHighlights[plr] then
                    local hl=Instance.new("Highlight")
                    hl.Adornee=plr.Character
                    hl.FillColor=Color3.fromRGB(0,0,255)
                    hl.OutlineColor=Color3.fromRGB(255,255,255)
                    hl.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop
                    hl.Parent=plr.Character
                    PlayerHighlights[plr]=hl
                end
            end
        end
    else
        for _,hl in pairs(PlayerHighlights) do if hl and hl.Parent then hl:Destroy() end end
        PlayerHighlights={}
    end

    -- ESP BrainHots
    if ESPBrainOn then
        for _,obj in pairs(workspace:GetDescendants()) do
            if obj:IsA("BasePart") and (obj.Name:lower():find("god") or obj.Name:lower():find("secret")) then
                if not BrainHighlights[obj] then
                    local hl=Instance.new("Highlight")
                    hl.Adornee=obj
                    hl.FillColor=Color3.fromRGB(255,0,0)
                    hl.OutlineColor=Color3.fromRGB(255,255,255)
                    hl.DepthMode=Enum.HighlightDepthMode.AlwaysOnTop
                    hl.Parent=obj
                    BrainHighlights[obj]=hl
                end
            end
        end
    else
        for _,hl in pairs(BrainHighlights) do if hl and hl.Parent then hl:Destroy() end end
        BrainHighlights={}
    end
end)

-- Aimbot teleguiado
local function GetClosestPlayer()
    local closest=nil
    local minDist=math.huge
    for _,plr in pairs(Players:GetPlayers()) do
        if plr~=LocalPlayer and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local dist=(plr.Character.HumanoidRootPart.Position-LocalPlayer.Character.HumanoidRootPart.Position).Magnitude
            if dist<minDist then
                minDist=dist
                closest=plr
            end
        end
    end
    return closest
end

UserInputService.InputBegan:Connect(function(input,gpe)
    if AimbotOn and input.UserInputType==Enum.UserInputType.Touch then
        local target=GetClosestPlayer()
        if target and target.Character then
            LocalPlayer.Character.HumanoidRootPart.CFrame=CFrame.new(LocalPlayer.Character.HumanoidRootPart.Position,target.Character.HumanoidRootPart.Position)
        end
    end
end)

-- Fly touch move
UserInputService.InputChanged:Connect(function(input)
    if FlyOn and input.UserInputType==Enum.UserInputType.Touch then
        local delta=input.Delta
        FlyVelocity = Vector3.new(delta.X/5,0,-delta.Y/5)
    end
end)
