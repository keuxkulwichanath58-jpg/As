-- ดึงข้อมูลตัวผู้เล่นและปุ่มกด
local player = game.Players.LocalPlayer
local button = script.Parent

-- กำหนดค่าเริ่มต้น
local isRunning = false
local normalSpeed = 16 -- ความเร็วปกติของ Roblox
local runSpeed = 50    -- ความเร็วตอนกดวิ่ง (ปรับแต่งตัวเลขได้ตามใจชอบ)

-- คำสั่งเมื่อมีคนคลิกที่ปุ่ม
button.MouseButton1Click:Connect(function()
    -- หาตัวละครของผู้เล่น
    local character = player.Character or player.CharacterAdded:Wait()
    local humanoid = character:WaitForChild("Humanoid")

    -- สลับสถานะเปิด-ปิด
    isRunning = not isRunning 

    -- เช็คสถานะเพื่อปรับความเร็ว
    if isRunning then
        humanoid.WalkSpeed = runSpeed
        button.Text = "วิ่งเร็ว: เปิด"
        button.BackgroundColor3 = Color3.fromRGB(50, 200, 50) -- เปลี่ยนปุ่มเป็นสีเขียว
    else
        humanoid.WalkSpeed = normalSpeed
        button.Text = "วิ่งเร็ว: ปิด"
        button.BackgroundColor3 = Color3.fromRGB(200, 50, 50) -- เปลี่ยนปุ่มเป็นสีแดง
    end
end)

