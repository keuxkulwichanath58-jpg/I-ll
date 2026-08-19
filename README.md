-- ==========================================
-- SCRIPT NAME: Akers UI System
-- Target Game: BlockSpin (Roblox)
-- ==========================================

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

local Window = Rayfield:CreateWindow({
   Name = "Akers Script | BlockSpin",
   LoadingTitle = "Akers System Loading...",
   LoadingSubtitle = "by Akers Developer",
   ConfigurationSaving = {
      Enabled = true,
      FolderName = "AkersConfig",
      FileName = "AkersBlockSpinSettings"
   },
   Discord = { Enabled = false },
   KeySystem = false
})

-- Global Variables & Services
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local Camera = workspace.CurrentCamera

-- Feature States
local WalkSpeedMultiplier = 1
local InfiniteJumpEnabled = false
local AutoFishEnabled = false
local InfiniteStaminaEnabled = false
local AntiLockEnabled = false
local ESP_Enabled = false
local FOV_Circle_Enabled = false
local FOV_Size = 90
local AimbotEnabled = false
local AimPart = "Head" -- "Head" or "HumanoidRootPart"

-- ------------------------------------------
-- Tab 1: Player Functions (ระบบผู้เล่น)
-- ------------------------------------------
local PlayerTab = Window:CreateTab("Player Settings", 4483362458)

-- 1. วิ่งเร็ว (x1 - x10)
PlayerTab:CreateSlider({
   Name = "WalkSpeed Multiplier (วิ่งเร็ว x1 - x10)",
   Range = {1, 10},
   Increment = 1,
   Suffix = "x",
   CurrentValue = 1,
   Flag = "SpeedSlider",
   Callback = function(Value)
      WalkSpeedMultiplier = Value
   end,
})

RunService.RenderStepped:Connect(function()
   if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
      LocalPlayer.Character.Humanoid.WalkSpeed = 16 * WalkSpeedMultiplier
   end
end)

-- 2. กระโดดไม่จำกัด (ไม่ติดวาป/เทเลพอร์ต)
PlayerTab:CreateToggle({
   Name = "Infinite Jump (กระโดดไม่จำกัด)",
   CurrentValue = false,
   Flag = "InfJumpToggle",
   Callback = function(Value)
      InfiniteJumpEnabled = Value
   end,
})

UserInputService.JumpRequest:Connect(function()
   if InfiniteJumpEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid") then
      LocalPlayer.Character:FindFirstChildOfClass("Humanoid"):ChangeState("Jumping")
   end
end)

-- 3. สเตมีน่าไม่จำกัด (ไม่ติดเทเลพอร์ต)
PlayerTab:CreateToggle({
   Name = "Infinite Stamina (สเตมีน่าไม่จำกัด)",
   CurrentValue = false,
   Flag = "InfStamToggle",
   Callback = function(Value)
      InfiniteStaminaEnabled = Value
   end,
})

task.spawn(function()
   while task.wait(0.2) do
      if InfiniteStaminaEnabled and LocalPlayer.Character then
         -- Bypass ค่า Stamina/Energy ในแมพ BlockSpin
         local char = LocalPlayer.Character
         local stam = char:FindFirstChild("Stamina") or char:FindFirstChild("Energy") or LocalPlayer:FindFirstChild("Stamina")
         if stam then
            stam.Value = 100
         end
      end
   end
end)

-- 4. กันล็อก + ศัตรูยิงไม่โดน (Anti-Lock / Desync)
PlayerTab:CreateToggle({
   Name = "Anti-Lock & Dodge (กันล็อก / ยิงไม่โดน)",
   CurrentValue = false,
   Flag = "AntiLockToggle",
   Callback = function(Value)
      AntiLockEnabled = Value
   end,
})

RunService.Heartbeat:Connect(function()
   if AntiLockEnabled and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
      local hrp = LocalPlayer.Character.HumanoidRootPart
      local realVelocity = hrp.Velocity
      -- ทำการเบี่ยงเบน Velocity ชั่วขณะเพื่อทำลายการคำนวณตำแหน่ง Aimbot ของผู้เล่นอื่น
      hrp.Velocity = Vector3.new(math.random(-150, 150), math.random(-50, 50), math.random(-150, 150))
      RunService.RenderStepped:Wait()
      hrp.Velocity = realVelocity
   end
end)


-- ------------------------------------------
-- Tab 2: Automation (ระบบอัตโนมัติ)
-- ------------------------------------------
local AutoTab = Window:CreateTab("Automation", 4483362458)

-- 5. ตกปลาอัตโนมัติ (Auto Fishing สำหรับ BlockSpin)
AutoTab:CreateToggle({
   Name = "Auto Fishing (ตกปลาอัตโนมัติ)",
   CurrentValue = false,
   Flag = "AutoFishToggle",
   Callback = function(Value)
      AutoFishEnabled = Value
   end,
})

task.spawn(function()
   while task.wait(0.3) do
      if AutoFishEnabled then
         pcall(function()
            local tool = LocalPlayer.Character:FindFirstChildOfClass("Tool")
            if tool and (tool.Name:lower():find("rod") or tool.Name:lower():find("fish")) then
               tool:Activate()
               -- กดคลิกอัตโนมัติส่งไปยังเซิร์ฟเวอร์
               game:GetService("VirtualUser"):Button1Down(Vector2.new(0,0))
               task.wait(0.1)
               game:GetService("VirtualUser"):Button1Up(Vector2.new(0,0))
            end
         end)
      end
   end
end)


-- ------------------------------------------
-- Tab 3: Visual & Combat (มองผู้เล่น / ล็อกเป้า)
-- ------------------------------------------
local VisualTab = Window:CreateTab("Visual & Aimbot", 4483362458)

-- สร้างรูปวงกลม FOV
local FOVCircle = Drawing.new("Circle")
FOVCircle.Thickness = 2
FOVCircle.NumSides = 60
FOVCircle.Radius = FOV_Size
FOVCircle.Filled = false
FOVCircle.Visible = false
FOVCircle.Color = Color3.fromRGB(0, 255, 120)

-- สไลเดอร์ปรับวงองศา (1-360)
VisualTab:CreateSlider({
   Name = "FOV Angle Circle (ปรับวง 1-360 องศา)",
   Range = {1, 360},
   Increment = 1,
   Suffix = "°",
   CurrentValue = 90,
   Flag = "FOVSlider",
   Callback = function(Value)
      FOV_Size = Value
      FOVCircle.Radius = FOV_Size
   end,
})

-- ช่องพิมพ์เลือกเลข (1-360)
VisualTab:CreateInput({
   Name = "Input FOV Degree (พิมพ์เลือกเลข 1-360)",
   PlaceholderText = "ป้อนตัวเลข 1-360...",
   RemoveTextAfterFocusLost = false,
   Callback = function(Text)
      local num = tonumber(Text)
      if num and num >= 1 and num <= 360 then
         FOV_Size = num
         FOVCircle.Radius = FOV_Size
      end
   end,
})

VisualTab:CreateToggle({
   Name = "Show FOV Circle (แสดงวงองศา)",
   CurrentValue = false,
   Flag = "FOVShowToggle",
   Callback = function(Value)
      FOV_Circle_Enabled = Value
      FOVCircle.Visible = Value
   end,
})

-- ตัวเลือกล็อกตัว / ล็อกหัว
VisualTab:CreateDropdown({
   Name = "Lock Target Part (ปรับล็อกตัว / ล็อกหัว)",
   Options = {"Head", "HumanoidRootPart"},
   CurrentOption = "Head",
   Flag = "TargetPartDropdown",
   Callback = function(Option)
      AimPart = Option
   end,
})

-- ปุ่มเปิด/ปิด ล็อกคนในวงองศา
VisualTab:CreateToggle({
   Name = "Aimbot Lock in FOV (ล็อกคนในวง)",
   CurrentValue = false,
   Flag = "AimbotToggle",
   Callback = function(Value)
      AimbotEnabled = Value
   end,
})

-- ESP มองผู้เล่น / ของในตัว / ของในกระเป๋า / รองรับคนเข้าใหม่
VisualTab:CreateToggle({
   Name = "Player & Backpack ESP (มองผู้เล่นและไอเทม)",
   CurrentValue = false,
   Flag = "ESPToggle",
   Callback = function(Value)
      ESP_Enabled = Value
   end,
})

-- ฟังก์ชั่นสร้าง ESP ติดตัวผู้เล่น
local function ApplyESP(plr)
   local function SetupCharacter(char)
      if not char then return end
      local head = char:WaitForChild("Head", 5)
      if not head then return end

      -- Highlight
      local highlight = char:FindFirstChild("Akers_ESP") or Instance.new("Highlight")
      highlight.Name = "Akers_ESP"
      highlight.Parent = char
      highlight.FillColor = Color3.fromRGB(255, 30, 30)
      highlight.OutlineColor = Color3.fromRGB(255, 255, 255)

      -- Billboard UI แสดงชื่อและของในกระเป๋า
      local billboard = head:FindFirstChild("Akers_Info") or Instance.new("BillboardGui")
      billboard.Name = "Akers_Info"
      billboard.Parent = head
      billboard.Size = UDim2.new(0, 200, 0, 50)
      billboard.StudsOffset = Vector3.new(0, 3, 0)
      billboard.AlwaysOnTop = true

      local textLabel = billboard:FindFirstChild("InfoLabel") or Instance.new("TextLabel")
      textLabel.Name = "InfoLabel"
      textLabel.Parent = billboard
      textLabel.Size = UDim2.new(1, 0, 1, 0)
      textLabel.BackgroundTransparency = 1
      textLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
      textLabel.TextStrokeTransparency = 0
      textLabel.TextSize = 13

      -- Loop อัปเดต ESP และไอเทมในกระเป๋า Real-time
      RunService.RenderStepped:Connect(function()
         if ESP_Enabled and char and char:Parent() and head:Parent() then
            highlight.Enabled = true
            billboard.Enabled = true
            
            -- ดึงรายการของที่ถืออยู่ + ของใน Backpack
            local items = {}
            if plr:FindFirstChild("Backpack") then
               for _, item in pairs(plr.Backpack:GetChildren()) do
                  if item:IsA("Tool") then table.insert(items, item.Name) end
               end
            end
            for _, item in pairs(char:GetChildren()) do
               if item:IsA("Tool") then table.insert(items, "[" .. item.Name .. "]") end
            end

            local itemListStr = #items > 0 and table.concat(items, ", ") or "ไม่มีของ"
            textLabel.Text = plr.Name .. "\n[ของในกระเป๋า: " .. itemListStr .. "]"
         else
            highlight.Enabled = false
            billboard.Enabled = false
         end
      end)
   end

   if plr.Character then SetupCharacter(plr.Character) end
   plr.CharacterAdded:Connect(SetupCharacter)
end

-- สแกนผู้เล่นปัจจุบัน และผู้เล่นที่เพิ่งเข้ามาใหม่ (Join Later)
for _, plr in pairs(Players:GetPlayers()) do
   if plr ~= LocalPlayer then
      ApplyESP(plr)
   end
end

Players.PlayerAdded:Connect(function(plr)
   if plr ~= LocalPlayer then
      ApplyESP(plr)
   end
end)

-- Main Loop สำหรับตำแหน่งวง FOV และระบบ Aimbot
RunService.RenderStepped:Connect(function()
   FOVCircle.Position = UserInputService:GetMouseLocation()

   if AimbotEnabled then
      local ClosestTarget = nil
      local ClosestDistance = FOV_Size

      for _, plr in pairs(Players:GetPlayers()) do
         if plr ~= LocalPlayer and plr.Character and plr.Character:FindFirstChild(AimPart) then
            local targetPart = plr.Character[AimPart]
            local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)

            if onScreen then
               local mousePos = UserInputService:GetMouseLocation()
               local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude

               if dist < ClosestDistance then
                  ClosestDistance = dist
                  ClosestTarget = targetPart
               end
            end
         end
      end

      if ClosestTarget then
         Camera.CFrame = CFrame.new(Camera.CFrame.Position, ClosestTarget.Position)
      end
   end
end)

-- แจ้งเตือนเมื่อโหลดสคริปต์เสร็จสมบูรณ์
Rayfield:Notify({
   Title = "Akers System Ready!",
   Content = "สคริปต์ Akers พร้อมใช้งานในแมพ BlockSpin แล้ว",
   Duration = 5,
   Image = 4483362458,
})
