local Rayfield = loadstring(game:HttpGet("https://sirius.menu/rayfield"))()
local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer

local Window = Rayfield:CreateWindow({
   Name = "Froggie Tycoon - Auto Collect Script",
   LoadingTitle = "Loading...",
   LoadingSubtitle = "By: Mitch",
   ConfigurationSaving = {
      Enabled = false,
      FolderName = nil, 
      FileName = "FroggieMitch"
   },
   Discord = {
      Enabled = false,
      Invite = "noinvitelink", 
      RememberJoins = true 
   },
   KeySystem = false, 
})

local MainTab = Window:CreateTab("Main", 4483362458)
local Section = MainTab:CreateSection("Configuration")

-- // VARIABLES
local SelectedPlotName = "Plot1"
local AutoFarm = false

-- // DROPDOWN: 6 Plots
local PlotDropdown = MainTab:CreateDropdown({
   Name = "Select Your Plot Number",
   Options = {"Plot1", "Plot2", "Plot3", "Plot4", "Plot5", "Plot6"},
   CurrentOption = {"Plot1"},
   MultipleOptions = false,
   Flag = "PlotSelect", 
   Callback = function(Option)
       SelectedPlotName = Option[1]
   end,
})

local StatusLabel = MainTab:CreateLabel("Status: Idle")

local Section2 = MainTab:CreateSection("Farming")

-- // HELPER: Get Position safely
local function getPosition(obj)
    if not obj then return nil end
    if obj:IsA("BasePart") then return obj.Position end
    if obj:IsA("Model") then return obj:GetPivot().Position end
    return nil
end

MainTab:CreateToggle({
   Name = "Auto Collect & Sell",
   CurrentValue = false,
   Flag = "AutoFarm", 
   Callback = function(Value)
        AutoFarm = Value
        
        task.spawn(function()
            while AutoFarm do
                local char = LocalPlayer.Character
                local hum = char and char:FindFirstChild("Humanoid")
                local root = char and char:FindFirstChild("HumanoidRootPart")
                
                -- 1. FIND PLOT
                local plot = Workspace:FindFirstChild("Plots") and Workspace.Plots:FindFirstChild(SelectedPlotName)
                
                if not plot then
                    StatusLabel:Set("Status: " .. SelectedPlotName .. " not found!")
                    task.wait(1)
                elseif hum and root then
                    
                    -- 2. FIND FOLDERS
                    local frogFolder = plot:FindFirstChild("SpawnedFrogs")
                    
                    -- Search recursively for ReleaseButton
                    local releaseBtn = nil
                    for _, obj in pairs(plot:GetDescendants()) do
                        if obj.Name == "ReleaseButton" then
                            releaseBtn = obj
                            break
                        end
                    end

                    -- 3. COLLECT FROGS
                    if frogFolder then
                        local frogs = frogFolder:GetChildren()
                        
                        if #frogs > 0 then
                            StatusLabel:Set("Status: Collecting " .. #frogs .. " Frogs...")
                            
                            for i, frog in pairs(frogs) do
                                if not AutoFarm then break end
                                
                                local pos = getPosition(frog)
                                if pos then
                                    hum:MoveTo(pos)
                                    
                                    local timer = 0
                                    repeat
                                        task.wait(0.1)
                                        timer = timer + 0.1
                                        if frog.Parent then hum:MoveTo(getPosition(frog)) end
                                        local dist = (root.Position - getPosition(frog)).Magnitude
                                    until dist < 5 or not frog.Parent or not AutoFarm or timer > 3
                                end
                            end
                            
                            -- 4. SELL
                            if AutoFarm and releaseBtn then
                                StatusLabel:Set("Status: Selling...")
                                local sellPos = getPosition(releaseBtn)
                                
                                if sellPos then
                                    hum:MoveTo(sellPos)
                                    
                                    local timer = 0
                                    repeat
                                        task.wait(0.1)
                                        timer = timer + 0.1
                                        hum:MoveTo(sellPos)
                                        local dist = (root.Position - sellPos).Magnitude
                                    until dist < 5 or not AutoFarm or timer > 8
                                    
                                    task.wait(1) -- Wait at button
                                else
                                    StatusLabel:Set("Status: Sell Button position error")
                                end
                            else
                                StatusLabel:Set("Status: Can't find ReleaseButton")
                            end
                            
                        else
                            StatusLabel:Set("Status: Waiting for frogs...")
                        end
                    else
                        StatusLabel:Set("Status: No frog folder in " .. SelectedPlotName)
                    end
                end
                
                task.wait(0.5)
            end
            
            StatusLabel:Set("Status: Stopped")
            -- Stop walking
            if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("Humanoid") then
                LocalPlayer.Character.Humanoid:MoveTo(LocalPlayer.Character.HumanoidRootPart.Position)
            end
        end)
   end,
})
