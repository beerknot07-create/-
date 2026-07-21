-- zxtata | Animal Hospital (Anomaly) Hub
local Players = game:GetService("Players")
local CoreGui = game:GetService("CoreGui")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")
local Camera = workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

if CoreGui:FindFirstChild("ZxtataAnimalHospital") then
    CoreGui.ZxtataAnimalHospital:Destroy()
end

local flying, fullbrightEnabled, espEnabled, speedEnabled = false, false, false, false
local flySpeed = 50
local walkSpeedVal = 25

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "ZxtataAnimalHospital"
ScreenGui.ResetOnSpawn = false
pcall(function() ScreenGui.Parent = CoreGui end)
if not ScreenGui.Parent then ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui") end

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 260, 0, 280)
MainFrame.Position = UDim2.new(0.5, -130, 0.3, -140)
MainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 15)
MainFrame.Active = true
MainFrame.Draggable = true
MainFrame.BorderSizePixel = 0
Instance.new("UICorner", MainFrame).CornerRadius = UDim.new(0, 10)

local TitleLabel = Instance.new("TextLabel", MainFrame)
TitleLabel.Size = UDim2.new(1, 0, 0, 38)
TitleLabel.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
TitleLabel.Text = "  zxtata | Animal Hospital Hub"
TitleLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
TitleLabel.Font = Enum.Font.SourceSansBold
TitleLabel.TextSize = 13
TitleLabel.BorderSizePixel = 0
Instance.new("UICorner", TitleLabel).CornerRadius = UDim.new(0, 10)

local function createToggle(text, yPos, callback)
    local btn = Instance.new("TextButton", MainFrame)
    btn.Size = UDim2.new(0, 220, 0, 38)
    btn.Position = UDim2.new(0, 20, 0, yPos)
    btn.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    btn.Text = text .. ": ปิดอยู่"
    btn.TextColor3 = Color3.fromRGB(255, 100, 100)
    btn.Font = Enum.Font.SourceSansBold
    btn.TextSize = 13
    btn.BorderSizePixel = 0
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    
    local state = false
    btn.MouseButton1Click:Connect(function()
        state = not state
        btn.Text = text .. (state and ": เปิดใช้งาน" or ": ปิดอยู่")
        btn.TextColor3 = state and Color3.fromRGB(100, 255, 100) or Color3.fromRGB(255, 100, 100)
        callback(state)
    end)
end

-- ฟังก์ชันเสริมช่วยเล่นในโรงพยาบาลสัตว์
createToggle("ระบบ บินสำรวจ (Fly V3)", 50, function(state)
    flying = state
    local char = LocalPlayer.Character
    if char and char:FindFirstChild("Humanoid") then
        char.Humanoid.PlatformStand = flying
    end
end)

createToggle("ระบบ วิ่งไวทันใจ (Speed)", 98, function(state)
    speedEnabled = state
end)

createToggle("ระบบ ส่องผู้เล่น/ตัวละคร (ESP)", 146, function(state)
    espEnabled = state
    if not espEnabled then
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Character and p.Character:FindFirstChild("HospitalESP") then
                p.Character.HospitalESP:Destroy()
            end
        end
    end
end)

createToggle("ระบบ เร่งแสงสว่าง (Fullbright)", 194, function(state)
    fullbrightEnabled = state
    if not fullbrightEnabled then
        Lighting.Brightness = 2
        Lighting.ClockTime = 14
        Lighting.GlobalShadows = true
    end
end)

-- ลูปการทำงานระบบ
RunService.Heartbeat:Connect(function()
    local char = LocalPlayer.Character
    if not char or not char:FindFirstChild("HumanoidRootPart") or not char:FindFirstChild("Humanoid") then return end
    local hrp = char.HumanoidRootPart
    local hum = char.Humanoid

    if speedEnabled then hum.WalkSpeed = walkSpeedVal else hum.WalkSpeed = 16 end

    if fullbrightEnabled then
        Lighting.Brightness = 3
        Lighting.ClockTime = 12
        Lighting.GlobalShadows = false
    end

    if espEnabled then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= LocalPlayer and p.Character then
                if not p.Character:FindFirstChild("HospitalESP") then
                    local hl = Instance.new("Highlight", p.Character)
                    hl.Name = "HospitalESP"
                    hl.FillColor = Color3.fromRGB(255, 50, 50)
                    hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                end
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
