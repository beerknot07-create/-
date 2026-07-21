-- zxtata | Ultimate Clean System
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

if CoreGui:FindFirstChild("ZxtataNewGui") then
    CoreGui.ZxtataNewGui:Destroy()
end

local flying, hitboxEnabled = false, false
local flySpeed = 50
local hitboxSize = 5

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ZxtataNewGui"
ScreenGui.ResetOnSpawn = false
pcall(function() ScreenGui.Parent = CoreGui end)
if not ScreenGui.Parent then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 260, 0, 150)
MainFrame.Position = UDim2.new(0.5, -130, 0.3, -75)
MainFrame.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 8)

local TitleLabel = Instance.new("TextLabel", MainFrame)
TitleLabel.Size = UDim2.new(1, 0, 0, 35)
TitleLabel.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
TitleLabel.Text = "  zxtata | New System"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.TextSize = 14
TitleLabel.BorderSizePixel = 0
Instance.new("UICorner", TitleLabel).CornerRadius = UDim.new(0, 8)

local FlyBtn = Instance.new("TextButton", MainFrame)
FlyBtn.Size = UDim2.new(0, 220, 0, 40)
FlyBtn.Position = UDim2.new(0, 20, 0, 45)
FlyBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
FlyBtn.Text = "ระบบ บิน V3: ปิดอยู่"
FlyBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
FlyBtn.Font = Enum.Font.SourceSansBold
FlyBtn.TextSize = 13
FlyBtn.BorderSizePixel = 0
Instance.new("UICorner", FlyBtn).CornerRadius = UDim.new(0, 6)

local HitboxBtn = Instance.new("TextButton", MainFrame)
HitboxBtn.Size = UDim2.new(0, 220, 0, 40)
HitboxBtn.Position = UDim2.new(0, 20, 0, 95)
HitboxBtn.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
HitboxBtn.Text = "ระบบ ขยาย Hitbox: ปิดอยู่"
HitboxBtn.TextColor3 = Color3.fromRGB(255, 100, 100)
HitboxBtn.Font = Enum.Font.SourceSansBold
HitboxBtn.TextSize = 13
HitboxBtn.BorderSizePixel = 0
Instance.new("UICorner", HitboxBtn).CornerRadius = UDim.new(0, 6)

FlyBtn.MouseButton1Click:Connect(function()
    flying = not flying
    FlyBtn.Text = flying and "ระบบ บิน V3: เปิดใช้งาน" or "ระบบ บิน V3: ปิดอยู่"
    FlyBtn.TextColor3 = flying and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.PlatformStand = flying
    end
end)

HitboxBtn.MouseButton1Click:Connect(function()
    hitboxEnabled = not hitboxEnabled
    HitboxBtn.Text = hitboxEnabled and "ระบบ ขยาย Hitbox: เปิดใช้งาน" or "ระบบ ขยาย Hitbox: ปิดอยู่"
    HitboxBtn.TextColor3 = hitboxEnabled and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
    if not hitboxEnabled then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
                p.Character.Head.Size = Vector3.new(2, 1, 1)
                p.Character.Head.Transparency = 0
                p.Character.Head.CanCollide = true
            end
        end
    end
end)

RunService.Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") or not char:FindFirstChild("Humanoid") then return end
    local hrp = char.HumanoidRootPart
    local hum = char.Humanoid

    if hitboxEnabled then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("Head") then
                local head = p.Character.Head
                head.Size = Vector3.new(hitboxSize, hitboxSize, hitboxSize)
                head.Transparency = 0.5
                head.CanCollide = false
            end
        end
    end

    if flying then
        hum.PlatformStand = true
        local camCFrame = Camera.CFrame
        local flyVector = Vector3.new(0, 0, 0)
        
        if UserInputService:IsKeyDown(Enum.KeyCode.W) then flyVector = flyVector + camCFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.S) then flyVector = flyVector - camCFrame.LookVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.D) then flyVector = flyVector + camCFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.A) then flyVector = flyVector - camCFrame.RightVector end
        if UserInputService:IsKeyDown(Enum.KeyCode.Space) then flyVector = flyVector + Vector3.new(0, 1, 0) end
        if UserInputService:IsKeyDown(Enum.KeyCode.LeftShift) then flyVector = flyVector - Vector3.new(0, 1, 0) end

        if flyVector.Magnitude > 0 then
            hrp.Velocity = flyVector.Unit * flySpeed
        else
            hrp.Velocity = Vector3.new(0, 0, 0)
        end
        hrp.CFrame = CFrame.new(hrp.Position, hrp.Position + camCFrame.LookVector)
    end
end)