--[[ 
    MARK SYSTEM: AUTO FARM & ESP (FIXED V.14)
    Admin: MARKW
    Features: Fixed Auto Coin, Role ESP, Persistent Webhook
]]

-- [1. CLEANUP UI]
local CoreGui = game:GetService("CoreGui")
if CoreGui:FindFirstChild("Rayfield") then CoreGui.Rayfield:Destroy() end

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

-- [2. CONFIG SYSTEM (จดจำ Webhook)]
local Config = { WebhookURL = "" }
local FileName = "MarkSystem_V14.json"

local function SaveConfig()
    writefile(FileName, HttpService:JSONEncode(Config))
end

if isfile(FileName) then
    Config = HttpService:JSONDecode(readfile(FileName))
end

-- [3. WEBHOOK FUNCTION]
local function SendStatus(title)
    if Config.WebhookURL ~= "" then
        local level = LocalPlayer:FindFirstChild("Level") or LocalPlayer:FindFirstChild("level")
        local data = {
            ["embeds"] = {{
                ["title"] = title,
                ["description"] = string.format(
                    "**ชื่อ:** %s\n**เลเวล:** %s\n**สถานะ:** กำลังทำงาน\n**Admin:** MARKW",
                    LocalPlayer.Name,
                    level and level.Value or "N/A"
                ),
                ["color"] = 16711680
            }}
        }
        request({
            Url = Config.WebhookURL,
            Method = "POST",
            Headers = {["Content-Type"] = "application/json"},
            Body = HttpService:JSONEncode(data)
        })
    end
end

-- [4. MAIN WINDOW]
local Window = Rayfield:CreateWindow({
   Name = "MARK SYSTEM | V.14 FIXED",
   LoadingTitle = "Admin: MARKW",
   LoadingSubtitle = "Auto Farm & ESP Rebuilt",
   ConfigurationSaving = { Enabled = false }
})

local Logic = { AutoCoin = false, ESP = false }

-- [5. FARM TAB - แก้ไขระบบเก็บเหรียญ]
local FarmTab = Window:CreateTab("Auto Farm", 4483362458)

FarmTab:CreateToggle({
   Name = "Auto Collect Coins (Smooth Float)",
   CurrentValue = false,
   Callback = function(Value)
      Logic.AutoCoin = Value
      task.spawn(function()
          while Logic.AutoCoin do
              pcall(function()
                  -- ค้นหา Container เหรียญ (ปรับให้ครอบคลุมหลายแมพ)
                  local CoinContainer = workspace:FindFirstChild("Normal") and workspace.Normal:FindFirstChild("CoinContainer") 
                                     or workspace:FindFirstChild("CoinContainer")
                  
                  if CoinContainer and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                      local Root = LocalPlayer.Character.HumanoidRootPart
                      
                      for _, coin in pairs(CoinContainer:GetChildren()) do
                          if Logic.AutoCoin and coin:IsA("BasePart") then
                              -- ระบบลอยเก็บ (ใช้ Tween เพื่อความนุ่มนวลป้องกันการเตะ)
                              local distance = (Root.Position - coin.Position).Magnitude
                              local info = TweenInfo.new(distance / 15, Enum.EasingStyle.Linear) -- ความเร็ว 15 studs/sec
                              local tween = TweenService:Create(Root, info, {CFrame = coin.CFrame * CFrame.new(0, 1, 0)})
                              
                              tween:Play()
                              tween.Completed:Wait()
                              task.wait(0.1) -- พักเล็กน้อยก่อนไปเหรียญถัดไป
                          end
                      end
                  end
              end)
              task.wait(1)
          end
      end)
   end,
})

-- [6. VISUAL TAB - ESP ออร่าแยกสี]
local VisualTab = Window:CreateTab("Visuals", 4483362458)

VisualTab:CreateToggle({
   Name = "Player Aura (ESP Roles)",
   CurrentValue = false,
   Callback = function(Value)
      Logic.ESP = Value
      task.spawn(function()
          while Logic.ESP do
              for _, p in pairs(Players:GetPlayers()) do
                  if p ~= LocalPlayer and p.Character then
                      local char = p.Character
                      local hl = char:FindFirstChild("MarkHighlight") or Instance.new("Highlight", char)
                      hl.Name = "MarkHighlight"
                      hl.FillTransparency = 0.5
                      
                      -- ตรวจสอบบทบาท
                      if p.Backpack:FindFirstChild("Knife") or char:FindFirstChild("Knife") then
                          hl.FillColor = Color3.fromRGB(255, 0, 0) -- ฆาตกร
                      elseif p.Backpack:FindFirstChild("Gun") or char:FindFirstChild("Gun") then
                          hl.FillColor = Color3.fromRGB(0, 0, 255) -- นายอำเภอ
                      else
                          hl.FillColor = Color3.fromRGB(0, 255, 0) -- ชาวบ้าน
                      end
                  end
              end
              task.wait(2)
          end
          -- ลบ ESP เมื่อปิด
          for _, p in pairs(Players:GetPlayers()) do
              if p.Character and p.Character:FindFirstChild("MarkHighlight") then
                  p.Character.MarkHighlight:Destroy()
              end
          end
      end)
   end,
})

-- [7. SETTINGS & WEBHOOK]
local SetTab = Window:CreateTab("Settings", 4483362458)

SetTab:CreateInput({
   Name = "Discord Webhook URL",
   PlaceholderText = "ลิ้งค์จะถูกจดจำอัตโนมัติ...",
   Callback = function(Text)
      Config.WebhookURL = Text
      SaveConfig()
      Rayfield:Notify({Title = "Saved", Content = "บันทึก Webhook แล้ว", Duration = 2})
   end,
})

SetTab:CreateButton({
   Name = "Test Webhook (ทดสอบระบบ)",
   Callback = function()
      SendStatus("MARK SYSTEM - Test Connection")
   end,
})

-- ตรวจสอบเมื่อจบเกม (โดยเช็คจากกระเป๋าหรือสถานะในเกม)
task.spawn(function()
    while true do
        task.wait(30)
        -- Logic: ถ้าไม่มีเหรียญในแมพแล้ว (จบตา) ให้ส่งสรุปผล
        local Coins = workspace:FindFirstChild("Normal") and workspace.Normal:FindFirstChild("CoinContainer")
        if Coins and #Coins:GetChildren() == 0 then
            SendStatus("จบเกมแล้ว - สรุปสถานะปัจจุบัน")
            task.wait(60) -- รอเริ่มตาใหม่
        end
    end
end)

SetTab:CreateLabel("MARKW - SYSTEM ADMIN")
