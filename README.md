--[[ 
    MARK SYSTEM: FARM & ESP EDITION (V.13)
    Admin: MARKW
    Features: Smooth Coin Farm, ESP Aura, Persistent Webhook
]]

-- [1. CLEANUP & INITIAL]
local CoreGui = game:GetService("CoreGui")
if CoreGui:FindFirstChild("Rayfield") then CoreGui.Rayfield:Destroy() end

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local LocalPlayer = Players.LocalPlayer
local HttpService = game:GetService("HttpService")

-- [2. PERSISTENT DATA SYSTEM]
local FileName = "MarkSystem_Config.json"
local Config = { WebhookURL = "" }

local function SaveConfig()
    writefile(FileName, HttpService:JSONEncode(Config))
end

local function LoadConfig()
    if isfile(FileName) then
        Config = HttpService:JSONDecode(readfile(FileName))
    end
end
LoadConfig()

-- [3. WEBHOOK SYSTEM]
local function SendWebhook(msg)
    if Config.WebhookURL ~= "" then
        local data = {
            ["content"] = "",
            ["embeds"] = {{
                ["title"] = "MARK SYSTEM - Game Summary",
                ["description"] = msg,
                ["color"] = 0x00ff00,
                ["footer"] = {["text"] = "Admin: MARKW"}
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

-- [4. LOADING PROGRESS]
for i = 1, 100 do task.wait(0.01) end

-- [5. MAIN WINDOW]
local Window = Rayfield:CreateWindow({
   Name = "MARK SYSTEM | FARM & ESP",
   LoadingTitle = "Admin: MARKW",
   LoadingSubtitle = "V.13 (No Key)",
   ConfigurationSaving = { Enabled = false }
})

-- Variables
local Logic = { AutoCoin = false, ESP = false }

-- [6. FARM TAB]
local FarmTab = Window:CreateTab("Auto Farm", 4483362458)

FarmTab:CreateToggle({
   Name = "Auto Collect Coins (Smooth Float)",
   CurrentValue = false,
   Callback = function(Value)
      Logic.AutoCoin = Value
      task.spawn(function()
          while Logic.AutoCoin do
              pcall(function()
                  local Root = LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart")
                  local Coins = workspace:FindFirstChild("Normal") and workspace.Normal:FindFirstChild("CoinContainer")
                  
                  if Root and Coins then
                      for _, coin in pairs(Coins:GetChildren()) do
                          if coin:IsA("BasePart") and Logic.AutoCoin then
                              -- ใช้การลอยไปหาแบบนุ่มนวล (ไม่วาร์ป) เพื่อป้องกันการโดนเตะ
                              repeat
                                  Root.CFrame = Root.CFrame:Lerp(coin.CFrame * CFrame.new(0, 2, 0), 0.2)
                                  task.wait()
                              until not coin.Parent or not Logic.AutoCoin
                          end
                      end
                  end
              end)
              task.wait(1)
          end
      end)
   end,
})

-- [7. VISUAL TAB (ESP)]
local VisualTab = Window:CreateTab("Visuals", 4483362458)

VisualTab:CreateToggle({
   Name = "Player Aura (ESP Roles)",
   CurrentValue = false,
   Callback = function(Value)
      Logic.ESP = Value
      task.spawn(function()
          while Logic.ESP do
              for _, p in pairs(Players:GetPlayers()) do
                  if p ~= LocalPlayer and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                      local char = p.Character
                      if not char:FindFirstChild("Highlight") then
                          local hl = Instance.new("Highlight", char)
                          hl.FillTransparency = 0.5
                          hl.OutlineTransparency = 0
                          
                          -- เช็คบทบาท (Logic เฉพาะของ MM2)
                          if p.Backpack:FindFirstChild("Knife") or char:FindFirstChild("Knife") then
                              hl.FillColor = Color3.fromRGB(255, 0, 0) -- ฆาตกร (แดง)
                          elseif p.Backpack:FindFirstChild("Gun") or char:FindFirstChild("Gun") then
                              hl.FillColor = Color3.fromRGB(0, 0, 255) -- นายอำเภอ (น้ำเงิน)
                          else
                              hl.FillColor = Color3.fromRGB(0, 255, 0) -- ชาวบ้าน (เขียว)
                          end
                      end
                  end
              end
              task.wait(2)
              if not Logic.ESP then
                  for _, p in pairs(Players:GetPlayers()) do
                      if p.Character and p.Character:FindFirstChild("Highlight") then
                          p.Character.Highlight:Destroy()
                      end
                  end
              end
          end
      end)
   end,
})

-- [8. WEBHOOK & SETTINGS]
local SetTab = Window:CreateTab("Settings", 4483362458)

SetTab:CreateInput({
   Name = "Discord Webhook URL",
   PlaceholderText = "วางลิ้งค์ Webhook ที่นี่...",
   Callback = function(Text)
      Config.WebhookURL = Text
      SaveConfig()
      Rayfield:Notify({Title = "Saved", Content = "จดจำ Webhook เรียบร้อยแล้ว", Duration = 2})
   end,
})

SetTab:CreateButton({
   Name = "Test Webhook",
   Callback = function()
      SendWebhook("ระบบทดสอบสถานะ: เชื่อมต่อสำเร็จ!\nชื่อ: " .. LocalPlayer.Name .. "\nเลเวล: " .. (LocalPlayer:FindFirstChild("level") and LocalPlayer.level.Value or "0"))
   end,
})

-- [9. AUTO SUMMARY (END GAME)]
-- ระบบนี้จะส่งข้อมูลเมื่อจบเกมอัตโนมัติ
task.spawn(function()
    while true do
        task.wait(10)
        -- เช็คสถานะจบเกมจากระบบของเกม (ตัวอย่าง)
        -- SendWebhook("จบเกม!\nชื่อ: " .. LocalPlayer.Name .. "\nเหรียญที่ได้: " .. coinCount)
    end
end)

SetTab:CreateLabel("MARKW - SYSTEM ADMIN")
