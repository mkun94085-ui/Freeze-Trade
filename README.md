--[[ 
    MARK SYSTEM: ULTIMATE FULL EDITION (NO KEY)
    Admin: MARKW
    Features: Realistic Loading (1-100%), Trade Manipulator, Visual Dupe
]]

local Rayfield = loadstring(game:HttpGet('https://sirius.menu/rayfield'))()

-- [SYSTEM: REALISTIC LOADING BAR]
local function StartLoading()
    Rayfield:Notify({
        Title = "MARK SYSTEM",
        Content = "กำลังเริ่มต้นระบบโดย Admin MARKW...",
        Duration = 2,
    })
    
    -- จำลองการโหลดทรัพยากร 1% - 100%
    for i = 1, 100 do
        local content = "กำลังดาวน์โหลดทรัพยากร: " .. i .. "%"
        if i == 100 then content = "ดาวน์โหลดทรัพยากรเสร็จสิ้น!" end
        
        -- อัปเดตสถานะการโหลด
        if i % 20 == 0 or i == 100 then
            Rayfield:Notify({
                Title = "MARK SYSTEM LOADING",
                Content = content,
                Duration = 1,
            })
        end
        task.wait(0.02) -- ความเร็วในการโหลด
    end
end

-- เริ่มกระบวนการโหลด
StartLoading()

-- [SYSTEM: MAIN WINDOW - ตัดระบบคีย์ออกแล้ว]
local Window = Rayfield:CreateWindow({
   Name = "MARK SYSTEM | PRIVATE V.9",
   LoadingTitle = "Accessing System: MARKW",
   LoadingSubtitle = "System Ready (No Key Required)",
   ConfigurationSaving = { Enabled = false }
})

-- Variables
local Players = game:GetService("Players")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local TradeRemote = ReplicatedStorage:WaitForChild("Trade"):WaitForChild("TradeCommand")
local Logic = { Freezing = false, AutoAccept = false, Target = nil }

-- 1. Main Tab: Trade Control
local MainTab = Window:CreateTab("Trade Control", 4483362458)

MainTab:CreateInput({
   Name = "Search Target",
   PlaceholderText = "Enter Username...",
   Callback = function(Text)
      for _, v in pairs(Players:GetPlayers()) do
         if v.Name:lower():match(Text:lower()) then
            Logic.Target = v
            Rayfield:Notify({Title = "System", Content = "Locked on: " .. v.Name, Duration = 2})
            break
         end
      end
   end,
})

MainTab:CreateSection("Visual System")

MainTab:CreateDropdown({
   Name = "Select Item",
   Options = {"Nik's Scythe", "Traveler's Axe", "Harvester", "Corrupt", "Icepiercer", "Batwing", "Candleflame"},
   CurrentOption = "Traveler's Axe",
   Callback = function(Option) _G.SelectedDupe = Option end,
})

MainTab:CreateButton({
   Name = "Execute Visual",
   Callback = function()
      if _G.SelectedDupe then
          Rayfield:Notify({
              Title = "Success",
              Content = "จำลอง " .. _G.SelectedDupe .. " สำเร็จ (Visual Only)",
              Duration = 4
          })
      end
   end,
})

-- 2. Exploits Tab: Freezer & Force Accept
local ExploitTab = Window:CreateTab("Exploits", 4483362458)

ExploitTab:CreateToggle({
   Name = "Freeze Trade",
   CurrentValue = false,
   Callback = function(Value)
      Logic.Freezing = Value
      if Value then
          local oldNamecall
          oldNamecall = hookmetamethod(game, "__namecall", function(self, ...)
              if self == TradeRemote and getnamecallmethod() == "FireServer" and Logic.Freezing then
                  return nil 
              end
              return oldNamecall(self, ...)
          end)
      end
   end,
})

ExploitTab:CreateToggle({
   Name = "Force Accept",
   CurrentValue = false,
   Callback = function(Value)
      Logic.AutoAccept = Value
      task.spawn(function()
          while Logic.AutoAccept do
              TradeRemote:FireServer("AcceptTrade")
              task.wait(0.1)
          end
      end)
   end,
})

-- 3. Settings Tab
local SetTab = Window:CreateTab("Settings", 4483362458)
SetTab:CreateDropdown({
   Name = "Language",
   Options = {"English", "Thai"},
   CurrentOption = "Thai",
   Callback = function(Option)
       Rayfield:Notify({Title = "System", Content = "Language set to " .. Option, Duration = 2})
   end,
})

SetTab:CreateLabel("MARKW - SYSTEM ADMIN")
