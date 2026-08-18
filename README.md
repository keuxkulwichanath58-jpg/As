-- [[ Akers Script - Murder vs Sheriff (9 Functions) ]] --

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Akers | Murder vs Sheriff",
   LoadingTitle = "Akers Hub Loading...",
   LoadingSubtitle = "by Akers Team",
   ConfigurationSaving = { Enabled = false }
})

local Tab = Window:CreateTab("Main Features", 4483362458)

-- Services & Variables
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse()

local ESP_Enabled = false
local AutoShootVisible_Enabled = false
local InfiniteJump_Enabled = false
local FOV_AutoShoot_Enabled = false
local ClickToLock_Enabled = false
local FOV_Size = 180
local WalkSpeed_Value = 16
local SelectedPlayerToTeleport = ""
local SelectedPlayerToKick = ""

-- วาดวงกลม FOV สีขาวกลางจอ
local FOVCircle = Drawing.new("Circle")
FOVCircle.Color = Color3.fromRGB(255, 255, 255)
FOVCircle.Thickness = 1.5
FOVCircle.NumSides = 64
FOVCircle.Filled = false
FOVCircle.Visible = false

-- 1. มองผู้เล่น (ESP)
Tab:CreateToggle({
   Name = "1. มองผู้เล่น (ESP)",
   CurrentValue = false,
   Callback = function(Value)
      ESP_Enabled = Value
      for _, player in pairs(Players:GetPlayers()) do
         if player ~= LocalPlayer and player.Character then
            if ESP_Enabled then
               if not player.Character:FindFirstChild("Akers_Highlight") then
                  local highlight = Instance.new("Highlight")
                  highlight.Name = "Akers_Highlight"
                  highlight.FillColor = Color3.fromRGB(255, 0, 0)
                  highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                  highlight.Parent = player.Character
               end
            else
               if player.Character:FindFirstChild("Akers_Highlight") then
                  player.Character.Akers_Highlight:Destroy()
               end
            end
         end
      end
   end,
})

-- 2. ยิงอัตโนมัติเมื่อเห็นตัว
Tab:CreateToggle({
   Name = "2. ยิงอัตโนมัติเมื่อเห็นตัว (Visible Check 100%)",
   CurrentValue = false,
   Callback = function(Value)
      AutoShootVisible_Enabled = Value
   end,
})

-- 3. กระโดดไม่จำกัด
Tab:CreateToggle({
   Name = "3. กระโดดไม่จำกัด (Infinite Jump)",
   CurrentValue = false,
   Callback = function(Value)
      InfiniteJump_Enabled = Value
   end,
})

UserInputService.JumpRequest:Connect(function()
   if InfiniteJump_Enabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
      LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
   end
end)

-- 4. ยิงอัตโนมัติในวง 360 องศา (มีวงสีขาวกลางจอ)
Tab:CreateToggle({
   Name = "4. ยิงอัตโนมัติในวง (FOV 360)",
   CurrentValue = false,
   Callback = function(Value)
      FOV_AutoShoot_Enabled = Value
      FOVCircle.Visible = Value
   end,
})

Tab:CreateSlider({
   Name = "ปรับขนาดวงกลม (1-360)",
   Range = {1, 360},
   Increment = 1,
   CurrentValue = 180,
   Callback = function(Value)
      FOV_Size = Value
      FOVCircle.Radius = Value
   end,
})

-- 5. วิ่งเร็ว (1-100)
Tab:CreateSlider({
   Name = "5. ปรับความเร็วการวิ่ง (WalkSpeed 1-100)",
   Range = {1, 100},
   Increment = 1,
   CurrentValue = 16,
   Callback = function(Value)
      WalkSpeed_Value = Value
   end,
})

-- 6. ล่องหน
Tab:CreateButton({
   Name = "6. ล่องหน (Invisibility)",
   Callback = function()
      if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("LowerTorso") then
         local root = LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
         if root then
            local clone = root:Clone()
            root:Destroy()
            clone.Parent = LocalPlayer.Character
            Rayfield:Notify({Title = "Akers", Content = "เปิดใช้งานล่องหนแล้ว", Duration = 3})
         end
      end
   end,
})

-- 7. วาปไปหาผู้เล่นที่เลือก
local PlayerDropdownTP = Tab:CreateDropdown({
   Name = "7. เลือกผู้เล่นเพื่อวาร์ป",
   Options = {"--- Select Player ---"},
   CurrentOption = {"--- Select Player ---"},
   Callback = function(Option) SelectedPlayerToTeleport = Option[1] end,
})

Tab:CreateButton({
   Name = "วาร์ปไปหาผู้เล่น",
   Callback = function()
      if SelectedPlayerToTeleport ~= "" and SelectedPlayerToTeleport ~= "--- Select Player ---" then
         local target = Players:FindFirstChild(SelectedPlayerToTeleport)
         if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character.HumanoidRootPart.CFrame = target.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
         end
      end
   end,
})

-- 8. กดหน้าจอกระสุนตรงล็อกเป้า (Click to Lock Bullet)
Tab:CreateToggle({
   Name = "8. กดหน้าจอกระสุนตรงล็อกเป้าฝั่งตรงข้าม",
   CurrentValue = false,
   Callback = function(Value)
      ClickToLock_Enabled = Value
   end,
})

Mouse.Button1Down:Connect(function()
   if ClickToLock_Enabled then
      local target = Mouse.Target
      if target and target.Parent and target.Parent:FindFirstChild("Humanoid") then
         local targetHead = target.Parent:FindFirstChild("Head")
         if targetHead then
            Camera.CFrame = CFrame.new(Camera.CFrame.Position, targetHead.Position)
            local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
            if tool then tool:Activate() end
         end
      end
   end
end)

-- 9. เตะผู้เล่นออกจากแมพ (Client-Side Drop/Freeze Attempt)
local PlayerDropdownKick = Tab:CreateDropdown({
   Name = "9. เลือกผู้เล่นเพื่อเตะออก",
   Options = {"--- Select Player ---"},
   CurrentOption = {"--- Select Player ---"},
   Callback = function(Option) SelectedPlayerToKick = Option[1] end,
})

Tab:CreateButton({
   Name = "เตะผู้เล่นที่เลือก",
   Callback = function()
      if SelectedPlayerToKick ~= "" and SelectedPlayerToKick ~= "--- Select Player ---" then
         local target = Players:FindFirstChild(SelectedPlayerToKick)
         if target and target.Character then
            target.Character:Destroy()
            Rayfield:Notify({Title = "Akers", Content = "ลบผู้เล่น "..SelectedPlayerToKick.." ออกจากจอของคุณแล้ว", Duration = 3})
         end
      end
   end,
})

-- ฟังก์ชันดึงรายชื่อผู้เล่นใส่ Dropdown
local function UpdatePlayerLists()
   local pList = {}
   for _, p in pairs(Players:GetPlayers()) do
      if p ~= LocalPlayer then table.insert(pList, p.Name) end
   end
   PlayerDropdownTP:Refresh(pList)
   PlayerDropdownKick:Refresh(pList)
end

Players.PlayerAdded:Connect(UpdatePlayerLists)
Players.PlayerRemoving:Connect(UpdatePlayerLists)
UpdatePlayerLists()

-- Render Loop หลัก
RunService.RenderStepped:Connect(function()
   FOVCircle.Position = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
   FOVCircle.Radius = FOV_Size

   if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
      LocalPlayer.Character.Humanoid.WalkSpeed = WalkSpeed_Value
   end

   if AutoShootVisible_Enabled or FOV_AutoShoot_Enabled then
      local ClosestTarget = nil
      local ShortestDistance = math.huge

      for _, player in pairs(Players:GetPlayers()) do
         if player ~= LocalPlayer and player.Character and player.Character:FindFirstChild("Head") and player.Character:FindFirstChild("Humanoid").Health > 0 then
            local head = player.Character.Head
            local pos, onScreen = Camera:WorldToViewportPoint(head.Position)

            if onScreen then
               local screenCenter = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2)
               local distance = (Vector2.new(pos.X, pos.Y) - screenCenter).Magnitude

               local isVisible = true
               if AutoShootVisible_Enabled then
                  local ray = Ray.new(Camera.CFrame.Position, (head.Position - Camera.CFrame.Position).Unit * 500)
                  local hit = workspace:FindPartOnRayWithIgnoreList(ray, {LocalPlayer.Character})
                  isVisible = hit and hit:IsDescendantOf(player.Character)
               end

               if isVisible then
                  if FOV_AutoShoot_Enabled and distance <= FOV_Size and distance < ShortestDistance then
                     ShortestDistance = distance
                     ClosestTarget = head
                  elseif AutoShootVisible_Enabled and distance < ShortestDistance then
                     ShortestDistance = distance
                     ClosestTarget = head
                  end
               end
            end
         end
      end

      if ClosestTarget then
         local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
         if tool then tool:Activate() end
      end
   end
end)

Rayfield:Notify({
   Title = "Akers Loaded!",
   Content = "สคริปต์ Akers 9 ฟังก์ชันพร้อมใช้งานแล้ว",
   Duration = 4,
   Image = 4483362458,
})
