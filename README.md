-- ตัวอย่างโครงสร้างสคริปต์พื้นฐาน (สำหรับศึกษาการเขียนโค้ดเท่านั้น)
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")

-- ตัวแปรสถานะฟังก์ชัน
local AimbotEnabled = false
local ESPEnabled = false
local SpeedEnabled = false
local MenuVisible = true

-- 1. ฟังก์ชันมองทะลุ (ESP - มองทุกคน)
local function ToggleESP(state)
    ESPEnabled = state
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            -- โค้ดสำหรับสร้างกรอบหรือไฮไลต์ตัวละครฝั่งตรงข้าม
            local highlight = player.Character:FindFirstChild("Highlight") or Instance.new("Highlight")
            highlight.Parent = player.Character
            highlight.Enabled = ESPEnabled
            highlight.FillColor = Color3.fromRGB(255, 0, 0) -- สีแดงสำหรับฝั่งตรงข้าม
        end
    end
end

-- 2. ฟังก์ชันล็อกเป้าฝั่งตรงข้าม (Aimbot)
local function GetClosestEnemy()
    local target = nil
    local shortestDist = math.huge
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Team ~= LocalPlayer.Team then
            local char = player.Character
            if char and char:FindFirstChild("Head") then
                local dist = (char.Head.Position - LocalPlayer.Character.Head.Position).Magnitude
                if dist < shortestDist then
                    shortestDist = dist
                    target = char.Head
                end
            end
        end
    end
    return target
end

RunService.RenderStepped:Connect(function()
    if AimbotEnabled then
        local targetHead = GetClosestEnemy()
        if targetHead and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            -- ปรับมุมมองกล้องไปที่หัวของฝั่งตรงข้าม
            workspace.CurrentCamera.CFrame = CFrame.new(workspace.CurrentCamera.CFrame.Position, targetHead.Position)
        end
    end

    -- 3. ฟังก์ชันวิ่งเร็ว (Speed Hack)
    if SpeedEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = 50 -- ความเร็วปกติมักจะอยู่ที่ 16
    elseif LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
        LocalPlayer.Character.Humanoid.WalkSpeed = 16
    end
end)

-- 4. ฟังก์ชันปุ่มปิด-เปิดเมนู (Toggle Key)
UserInputService.InputBegan:Connect(function(input, gameProcessed)
    if input.KeyCode == Enum.KeyCode.RightShift then -- ใช้ปุ่ม Right Shift เปิด-ปิด
        MenuVisible = not MenuVisible
        -- โค้ดแสดงหรือซ่อน UI เมนูหลัก
        print("Menu Visibility: " .. tostring(MenuVisible))
    end
end)
