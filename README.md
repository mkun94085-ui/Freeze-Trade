--[[ 
    MARK SYSTEM: ROLE ESP & AUTO FARM (FIXED ROLES V.15)
    Admin: MARKW
    Features: Advanced Role Scanner, Smooth Coin Farm, Persistent Webhook
]]

-- [1. CLEANUP & INIT]
local CoreGui = game:GetService("CoreGui")
if CoreGui:FindFirstChild("Rayfield") then CoreGui.Rayfield:Destroy() end

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

-- [2. PERSISTENT DATA (จดจำ Webhook)]
local Config = { WebhookURL = "" }
local FileName = "MarkSystem_V15.json"

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
                ["description"] = string.format("**ชื่อ:** %s\n**เลเวล:** %s\n**Admin:** MARKW", LocalPlayer.Name, level and level.Value or "N/A"),
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
   Name = "MARK SYSTEM | V.15 FIXED ROLES",
   LoadingTitle = "Admin: MARKW",
   LoadingSubtitle = "Role ESP & Farm Rebuilt",
   ConfigurationSaving = { Enabled = false }
})

local Logic = { AutoCoin = false, ESP = false }

-- [5. FARM TAB (AUTO COIN)]
local FarmTab = Window:CreateTab("Auto Farm", 4483362458)

FarmTab:CreateToggle({
   Name = "Auto Collect Coins (Smooth Float)",
   CurrentValue = false,
   Callback = function(Value)
      Logic.AutoCoin = Value
      task.spawn(function()
          while Logic.AutoCoin do
              pcall(function()
                  local CoinContainer = workspace:FindFirstChild("Normal") and workspace.Normal:FindFirstChild("CoinContainer")
                  if CoinContainer and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
                      local Root = LocalPlayer.Character.HumanoidRootPart
                      for _, coin in pairs(CoinContainer:GetChildren()) do
                          if Logic.AutoCoin and coin:IsA("BasePart") then
                              local distance = (Root.Position - coin.Position).Magnitude
                              local info = TweenInfo.new(distance / 16, Enum.EasingStyle.Linear)
                              local tween = TweenService:Create(Root, info, {CFrame = coin.CFrame * CFrame.new(0, 1.2, 0)})
                              tween:Play()
                              tween.Completed:Wait()
                              task.wait(0.05)
                          end
                      end
                  end
              end)
              task.wait(1)
          end
      end)
   end,
})

-- [6. VISUAL TAB (FIXED ROLE ESP)]
local VisualTab = Window:CreateTab("Visuals", 4483362458)

VisualTab:CreateToggle({
   Name = "Player Aura (Role Specific)",
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
                      hl.FillTransparency = 0.4
                      hl.OutlineTransparency = 0
                      
                      -- ระบบตรวจหาบทบาทที่แม่นยำขึ้น (ตรวจทั้งตัวและกระเป๋า)
                      local isMurderer = p.Backpack:FindFirstChild("Knife") or char:FindFirstChild("Knife")
                      local isSheriff = p.Backpack:FindFirstChild("Gun") or char:FindFirstChild("Gun")
                      
                      if isMurderer then
                          hl.FillColor = Color3.fromRGB(255, 0, 0) -- แดง (ฆาตกร)
                          hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                      elseif isSheriff then
                          hl.FillColor = Color3.fromRGB(0, 100, 255) -- น้ำเงิน (นายอำเภอ)
                          hl.OutlineColor = Color3.fromRGB(255, 255, 255)
                      else
                          hl.FillColor = Color3.fromRGB(0, 255, 100) -- เขียว (ชาวบ้าน)
                          hl.OutlineColor = Color3.fromRGB(0, 0, 0)
                      end
                  end
              end
              task.wait(1) -- อัปเดตสีทุก 1 วินาทีเพื่อความแม่นยำ
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
   PlaceholderText = "ระบบจดจำลิ้งค์ให้อัตโนมัติ...",
   Callback = function(Text)
      Config.WebhookURL = Text
      SaveConfig()
      Rayfield:Notify({Title = "Saved", Content = "บันทึกและจดจำ Webhook แล้ว", Duration = 2})
   end,
})

SetTab:CreateButton({
   Name = "Test Webhook (ทดสอบส่งข้อมูล)",
   Callback = function()
      SendStatus("MARK SYSTEM - เชื่อมต่อสำเร็จ!")
   end,
})

SetTab:CreateLabel("MARKW - SYSTEM ADMIN")
