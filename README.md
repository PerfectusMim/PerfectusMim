

if not game:IsLoaded() then
    game.Loaded:Wait()
end

local Rayfield = loadstring(game:HttpGet(('https://raw.githubusercontent.com/teehee567/Rayfield-Backupt/main/source')))()


local Window = Rayfield:CreateWindow({
	Name = "Anime Evolutions Simulator Private|PerfectusMim#0001",
	LoadingTitle = "Loading Perfectus|AES",
	LoadingSubtitle = "by Perfectus",
	ConfigurationSaving = {
		Enabled = true,
		FolderName = "Lcopium", -- Create a custom folder for your hub/game
		FileName = "Anime Evolution Simulator"
	},
        Discord = {
        	Enabled = true,
        	Invite = "37NZHy9gW3", -- The Discord invite code, do not include discord.gg/
        	RememberJoins = true -- Set this to false to make them join the discord every time they load it up
        },
	KeySystem = false, -- Set this to true to use our key system
})


game.Players.LocalPlayer.PlayerGui.UI.CenterFrame.Settings.Frame.MaxDistance.Frame.TextBox.Text = 10000

local services = require(game.Players.LocalPlayer.PlayerGui.UI.Client.Services)


for _,v in next, getconnections(services.CallEvent.Event) do
    if getfenv(v.Function).script.Name == "PunchingSettings" or getfenv(v.Function).script.Name == "DropSettings" or getfenv(v.Function).script.Name == "FrameSettings"then
        v:Disable()
    end
end


services.CallEvent.Event:Connect(function(yes)
    if yes[1] == "DropsCreate" then
        for i,v in pairs(yes[2]) do
            game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "DropCollect",[2] = i})
        end
    end
end)



local Tab1 = Window:CreateTab("READ", 4483362458)
Tab1:CreateParagraph({Title = "YOU STILL GET COINS AND POWER", Content = "Why cant i see drops? because they are autocollected and sent to server\nWhy cant i see damage/power? because they were removed for fps, otherwise you cant get past wave 200 in defence."})

Tab1:CreateToggle({
	Name = "Dont Show Drops/power (you WILL lag if off)",
	CurrentValue = false,
	Flag = "GS_dropscoinsdeath",
	Callback = function(Value)
        if Value then
            for _,v in next, getconnections(services.CallEvent.Event) do
                if getfenv(v.Function).script.Name == "PunchingSettings" or getfenv(v.Function).script.Name == "DropSettings" or getfenv(v.Function).script.Name == "FrameSettings"then
                    v:Disable()
                end
            end
        elseif not Value then
            for _,v in next, getconnections(services.CallEvent.Event) do
                if getfenv(v.Function).script.Name == "PunchingSettings" or getfenv(v.Function).script.Name == "DropSettings" or getfenv(v.Function).script.Name == "FrameSettings"then
                    v:Enable()
                end
            end
        end
	end,
})



local Tab2 = Window:CreateTab("Auto Farm", 4483362458)



-- -------------------------------------------------------------------------- --
--                                    Tab 1                                   --
-- -------------------------------------------------------------------------- --


-- ---------------------------------- basic --------------------------------- --

Tab2:CreateSection("Basic")

local collectdrops
function collectdrops_func()
    local collectdrops_conn
    collectdrops_conn = game.Workspace["__DROPS"].ChildAdded:Connect(function(child)
        if collectdrops then
            game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "DropCollect",[2] = child.name})
            child:Destroy()--literally what the game does, most likely how the original inf money exploit worked. Just dont delete it lol (doesnt work anymore)
        else
            collectdrops_conn:Disconnect()
        end
    end)
end

Tab2:CreateToggle({
	Name = "Collect Drops",
	CurrentValue = true,
	Flag = "Collect Drops",
	Callback = function(Value)
		collectdrops = Value
        if Value then
            collectdrops_func()
        end
	end,
})

local autoclickdebounce = false
local autoclick = false
local autoclicker = Tab2:CreateToggle({
	Name = "Autoclick (keep off when using the farms)",
	CurrentValue = false,
	Flag = "Autoclick",
	Callback = function(Value)
		autoclick = Value
		if Value then
            autoclickdebounce = false
            autoclick_func()
        end
	end,
})



function autoclick_func()
    spawn(function()
        while autoclick and wait(0.01) do
            if autoclickdebounce == false then
                firesignal(game.Players.LocalPlayer.PlayerGui.UI.BottomFrame.Click.TextButton.MouseButton1Click)
            end
        end
    end)
end



local power_delay
Tab2:CreateSlider({
	Name = "Speed Power Farm Delay",
	Range = {0, 01},
	Increment = 0.01,
	Suffix = "Seconds",
	CurrentValue = 0.01,
	Flag = "Speed Power Farm Delay",
	Callback = function(Value)
		power_delay = Value
	end,
})



local autopower
Tab2:CreateToggle({
	Name = "Speed Power Farm",
	CurrentValue = false,
	Flag = "Speed Power Farm",
	Callback = function(Value)
		autopower = Value
		if Value then
            autopower_func()
        end
	end,
})

function autopower_func()
    spawn(function()
        while autopower and wait(power_delay) do
            game:GetService("ReplicatedStorage").Remotes.Client:FireServer({"PowerTrain"})
        end
    end)
end







Tab2:CreateSection("Autofarm")

-- ------------------------------ Area Selector ----------------------------- --


Tab2:CreateParagraph({Title = "Auto Farm Area = Auto Defence)", Content = "Defence 5 included"})

local mobs = {}
local mobtofarm


local mobdropdown = Tab2:CreateDropdown({
	Name = "Mob List",
	Options = mobs,
	CurrentOption = "",
	Flag = "Star to open",
	Callback = function(Option)
        mobtofarm = Option
	end,
})


spawn(function() --i know, i know, open loop bad but i cant use .changed because its an upvalue
    local currentarea
    while wait(0.3) do
        if services.CurrentArea ~= currentarea then
	    mobs = {}
            mobdropdown:Refresh({})
            for _,mob in pairs(game.Workspace["__WORKSPACE"].Mobs[services.CurrentArea.Name]:GetChildren()) do
                if not table.find(mobs,mob.Name) then
                    table.insert(mobs,mob.name)
                end
            end
            mobdropdown:Refresh(mobs)
            currentarea = services.CurrentArea
        end
    end
end)

local autofarm
Tab2:CreateToggle({
	Name = "Auto Farm Selected Mob",
	CurrentValue = false,
	Flag = "Auto Farm Selected Mob",
	Callback = function(Value)
		autofarm = Value
        if Value then
            autofarm_func()
        end
	end,
})


function gettarget()
    local targetdistance = math.huge
    local target
    for _,mob in pairs(game.Workspace["__WORKSPACE"].Mobs[services.CurrentArea.Name]:GetChildren()) do
        if mob:IsA("Model") and mob.Name == mobtofarm then
            local distance = (game.Players.LocalPlayer.Character.HumanoidRootPart.Position - mob.HumanoidRootPart.Position).magnitude
            if distance < targetdistance and distance < 1000 then
                targetdistance = distance
                target = mob
            end
        end
    end
    return target
end

function autofarm_func()
    spawn(function()
        while wait(0.2) and autofarm do
            pcall(function()
                local target = gettarget()
                local hp = target.Settings.HP.Value
                if hp > 0 then
                    local oldhp
                    local failsafe = 0
                    local killfailsafe = false
                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = target.HumanoidRootPart.CFrame + Vector3.new(0, 2, 0)
                    spawn(function()
                        while wait() do
                            if failsafe > 3 then
                                autoclicker:Set(true)
                                if target.Settings.HP.Value > 0 then
                                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = target.HumanoidRootPart.CFrame + Vector3.new(0, 2, 0)
                                end
                                failsafe = 0
                            elseif killfailsafe then
                                return
                            end
                        end
                    end)
                    while hp > 0 and autofarm do
                        if autofarm then
                            oldhp = target.Settings.HP.Value
                            if hp == oldhp then
                                failsafe += 1
                            end
                            hp = target.Settings.HP.Value
                            if hp > 0 then
                                game:GetService("ReplicatedStorage").Remotes.Client:FireServer({'AttackMob', target, nil, 'left Arm'})
                                wait(0.1)
                            end
                        end
                    end
                    killfailsafe = true
                    return
                end
            end)
        end
    end)
end

-- -------------------------------- Autofarm -------------------------------- --

local autofarm2
Tab2:CreateToggle({
	Name = "Auto Farm Area",
	CurrentValue = false,
	Flag = "Auto Farm Area",
	Callback = function(Value)
		autofarm2 = Value
        if Value then
            autofarm_func2()
        end
	end,
})


function gettarget2()
    local targetdistance = math.huge
    local target
    for _,mob in pairs(game.Workspace["__WORKSPACE"].Mobs[services.CurrentArea.Name]:GetChildren()) do
        if mob:IsA("Model") then
            local distance = (game.Players.LocalPlayer.Character.HumanoidRootPart.Position - mob.HumanoidRootPart.Position).magnitude
            if distance < targetdistance and distance < 1000 then
                targetdistance = distance
                target = mob
            end
        end
    end
    return target
end

function autofarm_func2()
    autoclicker:Set(false)
    spawn(function()
        while wait(0.1) and autofarm2 do
            pcall(function()
                local target = gettarget2()
                local hp = target.Settings.HP.Value
                if hp > 0 then
                    local oldhp
                    local failsafe = 0
                    local killfailsafe = false
                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = target.HumanoidRootPart.CFrame + Vector3.new(0, 2, 0)
                    spawn(function()
                        while wait() do
                            if failsafe > 3 then
                                if target.Settings.HP.Value > 0 then
                                    game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = target.HumanoidRootPart.CFrame + Vector3.new(0, 2, 0)
                                end
                                failsafe = 0
                            elseif killfailsafe then
                                return
                            end
                        end
                    end)
                    while hp > 0 and autofarm2 do
                        if autofarm2 then
                            oldhp = target.Settings.HP.Value
                            if hp == oldhp then
                                failsafe += 1
                            end
                            hp = target.Settings.HP.Value
                            if hp > 0 then
                                game:GetService("ReplicatedStorage").Remotes.Client:FireServer({'AttackMob', target, nil, 'left Arm'})
                                game:GetService("RunService").RenderStepped:Wait()
                            end
                        end
                    end
                    killfailsafe = true
                    return
                end
            end)
        end
    end)
end








-- -------------------------------------------------------------------------- --
--                                Auto Defence                                --
-- -------------------------------------------------------------------------- --










-- -------------------------------------------------------------------------- --
--                                Special Farm                                --
-- -------------------------------------------------------------------------- --







-- -------------------------------------------------------------------------- --
--                                  Auto Open                                 --
-- -------------------------------------------------------------------------- --



local Tab5 = Window:CreateTab("Auto open", 4483362458)


Tab5:CreateSection("Opener")


local eggs ={}

for _,v in pairs(game.Workspace["__WORKSPACE"].FightersPoint:GetChildren()) do
    table.insert(eggs,v.Name)
end

local eggtoopen = "iagree"
Tab5:CreateDropdown({
	Name = "Star to open",
	Options = eggs,
	CurrentOption = "",
	Flag = "Star to open",
	Callback = function(Option)
        eggtoopen = Option
        getfighterinstars()
	end,
})

local openegg
Tab5:CreateToggle({
	Name = "Open Stars",
	CurrentValue = false,
	Flag = "Open Stars",
	Callback = function(Value)
		openegg = Value
        if Value then
            spawn(function()
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = game.Workspace["__WORKSPACE"].FightersPoint[eggtoopen]["W-1"].CFrame
                openegg_func()
            end)
        end
	end,
})




function openegg_func()
    spawn(function()
        while wait(0.5) and openegg do
            local data = {
                [1]="BuyTier",
                [2]=game.Workspace["__WORKSPACE"].FightersPoint[eggtoopen],
                [3]="E",
                [4]={}
            }
            services.PlayerData.Gamepasses["Triple Fighters"] = true
            game.ReplicatedStorage.Remotes.Client:FireServer(data)
        end
    end)
end



-- ------------------------------- Auto Delete ------------------------------ --



Tab5:CreateSection("Auto Delete")
Tab5:CreateParagraph({Title = "Death", Content = "Only deletes new fighters from opening"})

local fighters = require(game:GetService("ReplicatedStorage").Modules.Fighters)
local fighterinstar = {}
local dontkillthisone1
local dontedelete1 = Tab5:CreateDropdown({
	Name = "Fighter to not autodelete",
	Options = fighterinstar,
	CurrentOption = "",
	Flag = "Fighter to not autodelete",
	Callback = function(Option)
        dontkillthisone1 = Option
	end,
})



function getfighterinstars()
    for _,v in pairs(game:GetService("Workspace")["__WORKSPACE"].FightersPoint:GetChildren()) do
        if v.Name == eggtoopen then
            local lol = {}
            dontedelete1:Refresh({})
            table.insert(lol,"Nothing")
            for _,v1 in pairs(v.Fighters:GetChildren()) do
                table.insert(lol,v1.Name)
            end
            dontedelete1:Refresh(lol)
            break
        end
    end
end



local delestar1
Tab5:CreateToggle({
	Name = "Delete 1 Stars",
	CurrentValue = false,
	Flag = "Delete 1 Stars",
	Callback = function(Value)
		delestar1 = Value
	end,
})

function deletestar(unit, star, pain)
    if unit ~= dontkillthisone1 then
        if fighters[unit].Stars == star then
            local data = {
                [1] = "EquipFighter",
                [2] = "Delete",
                [3] = {[pain] = true}
            }
            game.ReplicatedStorage.Remotes.Client:FireServer(data)
        end
    end
end




local delestar2
Tab5:CreateToggle({
	Name = "Delete 2 Stars",
	CurrentValue = false,
	Flag = "Delete 2 Stars",
	Callback = function(Value)
		delestar2 = Value
	end,
})


local delestar3
Tab5:CreateToggle({
	Name = "Delete 3 Stars",
	CurrentValue = false,
	Flag = "Delete 3 Stars",
	Callback = function(Value)
		delestar3 = Value
	end,
})


local delestar4
Tab5:CreateToggle({
	Name = "Delete 4 Stars",
	CurrentValue = false,
	Flag = "Delete 4 Stars",
	Callback = function(Value)
		delestar4 = Value
	end,
})
local delestar5
Tab5:CreateToggle({
	Name = "Delete 5 Stars",
	CurrentValue = false,
	Flag = "Delete 5 Stars",
	Callback = function(Value)
		delestar5 = Value
	end,
})


game.Players.LocalPlayer.PlayerGui.UI.CenterFrame.Backpack.Frame.ChildAdded:Connect(function(unit) --  yea.. i cbf to optimise this shit
    if unit:IsA("ImageLabel") and unit.Frame.ViewportFrame:FindFirstChildWhichIsA("Model") then
        local unitname = unit.Frame.ViewportFrame:FindFirstChildWhichIsA("Model").Name
        if delestar1 then
            deletestar(unitname, 1, unit.name)
        end
        if delestar2 then
            deletestar(unitname, 2, unit.name)
        end
        if delestar3 then
            deletestar(unitname, 3, unit.name)
        end
        if delestar4 then
            deletestar(unitname, 4, unit.name)
        end
        if delestar5 then
            deletestar(unitname, 5, unit.name)
        end
    end
end)

pcall(function()
    game.Players.LocalPlayer.PlayerGui.UI.Client.Modules.AnimationSettings:Destroy()
    game.Players.LocalPlayer.PlayerGui.Animation:Destroy()
end)



-- -------------------------------------------------------------------------- --
--                                  Teleports                                 --
-- -------------------------------------------------------------------------- --




local Tab6 = Window:CreateTab("Teleports", 4483362458)



for _,v in pairs(game:GetService("Workspace")["__WORKSPACE"].Areas:GetChildren()) do
    Tab6:CreateButton({
        Name = v.name,
        Callback = function()
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v.Point.CFrame
        end,
    })
end

-- ----------------------------- area teleports ----------------------------- --
local Tab11 = Window:CreateTab("Power Teleports", 4483362458)

for _,v in pairs(game:GetService("Workspace")["__WORKSPACE"].Useless:GetChildren()) do
    if v.name == "x2Area" then
        Tab11:CreateButton({
            Name = v["W-1"].Gui.Title.Text,
            Callback = function()
                game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = v["W-1"].CFrame
            end,
        })
    end
end

-- -------------------------------------------------------------------------- --
--                                 Open Menus                                 --
-- -------------------------------------------------------------------------- --


local Tab7 = Window:CreateTab("Menus", 4483362458)
Tab7:CreateSection("Main GUIS")

Tab7:CreateButton({
	Name = "Fuse",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Fuse"})
	end,
})
Tab7:CreateButton({
	Name = "Teleport",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Teleport"})
	end,
})
Tab7:CreateButton({
	Name = "SSystem",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "SSystem"})
	end,
})
Tab7:CreateButton({
	Name = "Enchants",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Enchants"})
	end,
})
Tab7:CreateButton({
	Name = "Grimoires",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Grimoires"})
	end,
})
Tab7:CreateButton({
	Name = "Exchange",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Exchange"})
	end,
})
Tab7:CreateButton({
	Name = "Passive",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Passive"})
	end,
})
Tab7:CreateButton({
	Name = "Stat Upgrade",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Status"})
	end,
})
Tab7:CreateButton({
	Name = "Avatars",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Avatars"})
	end,
})
Tab7:CreateButton({
	Name = "Classes",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Class"})
	end,
})

Tab7:CreateSection("Less Useful GUIS")
Tab7:CreateButton({
	Name = "Rank Up",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Rank Up"})
	end,
})
Tab7:CreateButton({
	Name = "Auras",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Auras"})
	end,
})
Tab7:CreateButton({
	Name = "Weapons",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Weapons"})
	end,
})
Tab7:CreateButton({
	Name = "Limit Break",
	Callback = function()
		services.CallEvent:Fire({"OpenFrame", "Limit Break"})
	end,
})


-- -------------------------------------------------------------------------- --
--                                Miscellaneous                               --
-- -------------------------------------------------------------------------- --
local Tab8 = Window:CreateTab("Misc", 4483362458)
Tab8:CreateSection("Weird Stuff")

local autogift
function autogift_func()
    spawn(function()
        while wait(10) and autogift do
            for _,v in pairs(game.Players.LocalPlayer.PlayerGui.UI.CenterFrame.Gifts.Frame:GetChildren()) do
                if v:IsA("ImageLabel") and v.Frame.TextLabel.Text == "Claim" then
                    game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "Gift",[2] = v.Name})                    
                end
            end
        end
    end)
end


Tab8:CreateToggle({
	Name = "Auto Gifts",
	CurrentValue = true,
	Flag = "Auto Gifts",
	Callback = function(Value)
		autogift = Value
        if Value then
            autogift_func()
        end
	end,
})

Tab8:CreateLabel("Anti-Afk (Chill my guy its already on)")

Tab8:CreateSection("FPS Boost")


local framesettings = require(game.Players.LocalPlayer.PlayerGui.UI.Client.Modules.FrameSettings)
Tab8:CreateButton({
	Name = "FPS Boost",
	Callback = function()
		fps_booster()
        framesettings.CreateDropPopUp = function() return end
        for _,v in pairs(game:GetService("ReplicatedStorage").Assets.Template.Effect:GetChildren()) do
            v:Destroy()
        end
	end,
})








function fps_booster()
    _G.Settings = {
        Players = {
            ["Ignore Me"] = true, -- Ignore your Character
            ["Ignore Others"] = true-- Ignore other Characters
        },
        Meshes = {
            Destroy = false, -- Destroy Meshes
            LowDetail = true -- Low detail meshes (NOT SURE IT DOES ANYTHING)
        },
        Images = {
            Invisible = true, -- Invisible Images
            LowDetail = false, -- Low detail images (NOT SURE IT DOES ANYTHING)
            Destroy = false, -- Destroy Images
        },
        ["No Particles"] = true, -- Disables all ParticleEmitter, Trail, Smoke, Fire and Sparkles
        ["No Camera Effects"] = true, -- Disables all PostEffect's (Camera/Lighting Effects)
        ["No Explosions"] = true, -- Makes Explosion's invisible
        ["No Clothes"] = false, -- Removes Clothing from the game
        ["Low Water Graphics"] = true, -- Removes Water Quality
        ["No Shadows"] = true, -- Remove Shadows
        ["Low Rendering"] = true, -- Lower Rendering
        ["Low Quality Parts"] = true -- Lower quality parts
    }



    local Players = game:GetService("Players")
    local BadInstances = {"DataModelMesh", "FaceInstance", "ParticleEmitter", "Trail", "Smoke", "Fire", "Sparkles", "PostEffect", "Explosion", "Clothing", "BasePart"}
    local CanBeEnabled = {"ParticleEmitter", "Trail", "Smoke", "Fire", "Sparkles", "PostEffect"}
    local function PartOfCharacter(Instance)
        for i, v in pairs(Players:GetPlayers()) do
            if v.Character and Instance:IsDescendantOf(v.Character) then
                return true
            end
        end
        return false
    end
    local function ReturnDescendants()
        local Descendants = {}
        WaitNumber = 5000
        if _G.Settings.Players["Ignore Others"] then
            for i, v in pairs(game:GetDescendants()) do
                if not v:IsDescendantOf(Players) and not PartOfCharacter(v) then
                    for i2, v2 in pairs(BadInstances) do
                        if v:IsA(v2) then
                            table.insert(Descendants, v)
                        end
                    end
                end
                if i == WaitNumber then
                    task.wait()
                    WaitNumber = WaitNumber + 5000
                end
            end
        elseif _G.Settings.Players["Ignore Me"] then
            for i, v in pairs(game:GetDescendants()) do
                if not v:IsDescendantOf(Players) and not v:IsDescendantOf(ME.Character) then
                    for i2, v2 in pairs(BadInstances) do
                        if v:IsA(v2) then
                            table.insert(Descendants, v)
                        end
                    end
                end
                if i == WaitNumber then
                    task.wait()
                    WaitNumber = WaitNumber + 5000
                end
            end
        end
        return Descendants
    end
    local function CheckIfBad(Instance)
        if not Instance:IsDescendantOf(Players) and not PartOfCharacter(Instance) then
            if Instance:IsA("DataModelMesh") then
                if _G.Settings.Meshes.LowDetail then
                    sethiddenproperty(Instance, "LODX", Enum.LevelOfDetailSetting.Low)
                    sethiddenproperty(Instance, "LODY", Enum.LevelOfDetailSetting.Low)
                elseif _G.Settings.Meshes.Destroy then
                    Instance:Destroy()
                end
            elseif Instance:IsA("FaceInstance") then
                if _G.Settings.Images.Invisible then
                    Instance.Transparency = 1
                elseif _G.Settings.Images.LowDetail then
                    Instance.Shiny = 1
                elseif _G.Settings.Images.Destroy then
                    Instance:Destroy()
                end
            elseif table.find(CanBeEnabled, Instance.ClassName) then
                if _G.Settings["No Particles"] or (_G.Settings.Other and _G.Settings.Other["No Particles"]) then
                    Instance.Enabled = false
                end
            elseif Instance:IsA("Explosion") then
                if _G.Settings["No Explosions"] or (_G.Settings.Other and _G.Settings.Other["No Explosions"]) then
                    Instance.Visible = false
                end
            elseif Instance:IsA("Clothing") then
                if _G.Settings["No Clothes"] or (_G.Settings.Other and _G.Settings.Other["No Clothes"]) then
                    Instance:Destroy()
                end
            elseif Instance:IsA("BasePart") then
                if _G.Settings["Low Quality Parts"] or (_G.Settings.Other and _G.Settings.Other["Low Quality Parts"]) then
                    Instance.Material = Enum.Material.Plastic
                    Instance.Reflectance = 0
                end
            end
        end
    end
    if _G.Settings["Low Water Graphics"] or (_G.Settings.Other and _G.Settings.Other["Low Water Graphics"]) then
        workspace:FindFirstChildOfClass("Terrain").WaterWaveSize = 0
        workspace:FindFirstChildOfClass("Terrain").WaterWaveSpeed = 0
        workspace:FindFirstChildOfClass("Terrain").WaterReflectance = 0
        workspace:FindFirstChildOfClass("Terrain").WaterTransparency = 0
    end
    if _G.Settings["No Shadows"] or (_G.Settings.Other and _G.Settings.Other["No Shadows"]) then
        game:GetService("Lighting").GlobalShadows = false
        game:GetService("Lighting").FogEnd = 9e9
    end
    if _G.Settings["Low Rendering"] or (_G.Settings.Other and _G.Settings.Other["Low Rendering"]) then
        settings().Rendering.QualityLevel = 1
    end
    local Descendants = ReturnDescendants()
    local WaitNumber = 500
    for i, v in pairs(Descendants) do
        CheckIfBad(v)
        if i == WaitNumber then
            task.wait()
            WaitNumber = WaitNumber + 500
        end
    end
    Rayfield:Notify({
        Title = "FPS Booster",
        Content = "Loaded",
        Duration = 6.5,
        Image = 4483362458,
        Actions = { -- Notification Buttons
        },
    })
    game.DescendantAdded:Connect(CheckIfBad)
end





-- -------------------------------- Auto Buy -------------------------------- --

Tab8:CreateSection("Auto Buy")


local autoaura
local autorank
local autoweapon

Tab8:CreateToggle({
	Name = "Auto Aura",
	CurrentValue = false,
	Flag = "Auto Aura",
	Callback = function(Value)
		autoaura = Value
        if Value then
            autoaura_func()
        end
	end,
})
Tab8:CreateToggle({
	Name = "Auto Rank",
	CurrentValue = false,
	Flag = "Auto Rank",
	Callback = function(Value)
		autorank = Value
        if Value then
            autorank_func()
        end
	end,
})
Tab8:CreateToggle({
	Name = "Auto Weapon",
	CurrentValue = false,
	Flag = "Auto Weapon",
	Callback = function(Value)
		autoweapon = Value
        if Value then
            autoweapon_func()
        end
	end,
})



local OldNameCall = nil
OldNameCall = hookmetamethod(game, "__namecall", function(Self, ...)
    local NamecallMethod = getnamecallmethod()

    if NamecallMethod == "Kick" then
        return nil
    end

    return OldNameCall(Self, ...)
end)

local auras = require(game.ReplicatedStorage.Modules.Auras)
local ranks = require(game.ReplicatedStorage.Modules.Ranks)
local weapons = require(game.ReplicatedStorage.Modules.Weapons)

function autoaura_func()
    while wait(1) and autoaura do
        if services.PlayerData.Coins > auras[services.PlayerData.CurrentAura+1].CoinsCost and autoaura then
            game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "Aura"})
        end
    end
end

function autorank_func()
    while wait(1) and autorank do
        if services.PlayerData.Coins > ranks[services.PlayerData.CurrentRank+1].CoinsCost and services.PlayerData.Power > ranks[services.PlayerData.CurrentRank+1].PowerCost and autorank then
            game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "RankUp"})
        end
    end
end

function autoweapon_func()
    while wait(1) and autoweapon do
        if services.PlayerData.Power > weapons[services.PlayerData.CurrentWeapon[1]+1].PowerRequires and autoweapon then
            local savepoint = game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame
            game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "Weapon"})
            wait(3)
            game.Players.LocalPlayer.Character.HumanoidRootPart.CFrame = savepoint
        end
    end
end







-- -------------------------------- Movement -------------------------------- --


Tab8:CreateSection("Movement")
local walkspeed
local delrunning = false
Tab8:CreateSlider({
	Name = "Walk Speed (0 for default running)",
	Range = {0, 500},
	Increment = 10,
	Suffix = "",
	CurrentValue = 0,
	Flag = "Walk Speed",
	Callback = function(Value)
		walkspeed = Value
        if Value ~= 0 then
            delrunning = true
            game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = Value
        end
	end,
})


spawn(function() -- sorry for dumb code but im honestly clueless because of the way the character model works
    while wait(5) do
        if delrunning then
            game.Players.LocalPlayer.Character.Humanoid.WalkSpeed = walkspeed
            pcall(function() game.Workspace:FindFirstChild("Running",1):Destroy() end)
        end
    end
end)

-- -------------------------------------------------------------------------- --
--                                   AutoUse                                  --
-- -------------------------------------------------------------------------- --


local Tab9 = Window:CreateTab("Auto Use", 4483362458)

Tab9:CreateSection("Auto Boost")

Tab9:CreateToggle({
	Name = "Toggle all auto Boost",
	CurrentValue = false,
	Flag = "Toggle all auto Boost",
	Callback = function(Value)
        if Value then
            lucky_tog:Set(true)
            coins_tog:Set(true)
            power_tog:Set(true)
            gems_tog:Set(true)
            damage_tog:Set(true)
        else
            lucky_tog:Set(false)
            coins_tog:Set(false)
            power_tog:Set(false)
            gems_tog:Set(false)
            damage_tog:Set(false)
        end
	end,
})


local x2lucky
lucky_tog = Tab9:CreateToggle({
	Name = "Auto Use X2Lucky",
	CurrentValue = false,
	Flag = "Auto Use X2Lucky",
	Callback = function(Value)
		x2lucky = Value
        if Value then
            spawn(function()
                while wait(1) and x2lucky do
                    if services.PlayerData.Boosts["x2 Lucky"][1] <= 0 and services.PlayerData.Boosts["x2 Lucky"][2] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "BoostUse",[2] = "x2 Lucky"})
                    end
                end
            end)
        end
	end,
})


local x2coins
coins_tog = Tab9:CreateToggle({
	Name = "Auto Use X2Coins",
	CurrentValue = false,
	Flag = "Auto Use X2Coins",
	Callback = function(Value)
		x2coins = Value
        if Value then
            spawn(function()
                while wait(1) and x2coins do
                    if services.PlayerData.Boosts["x2 Coins"][1] <= 0 and services.PlayerData.Boosts["x2 Coins"][2] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "BoostUse",[2] = "x2 Coins"})
                    end
                end
            end)
        end
	end,
})



local x2power
power_tog = Tab9:CreateToggle({
	Name = "Auto Use X2Power",
	CurrentValue = false,
	Flag = "Auto Use X2Power",
	Callback = function(Value)
		x2power = Value
        if Value then
            spawn(function()
                while wait(1) and x2power do
                    if services.PlayerData.Boosts["x2 Power"][1] <= 0 and services.PlayerData.Boosts["x2 Power"][2] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "BoostUse",[2] = "x2 Power"})
                    end
                end
            end)
        end
	end,
})


local x2gems
power_tog = Tab9:CreateToggle({
	Name = "Auto Use X2Gems",
	CurrentValue = false,
	Flag = "Auto Use X2Gems",
	Callback = function(Value)
		x2gems = Value
        if Value then
            spawn(function()
                while wait(1) and x2gems do
                    if services.PlayerData.Boosts["x2 Gems"][1] <= 0 and services.PlayerData.Boosts["x2 Gems"][2] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "BoostUse",[2] = "x2 Gems"})
                    end
                end
            end)
        end
	end,
})


local x2damage
damage_tog = Tab9:CreateToggle({
	Name = "Auto Use X2Damage",
	CurrentValue = false,
	Flag = "Auto Use X2Damage",
	Callback = function(Value)
		x2damage = Value
        if Value then
            spawn(function()
                while wait(1) and x2damage do
                    if services.PlayerData.Boosts["x2 Damage"][1] <= 0 and services.PlayerData.Boosts["x2 Damage"][2] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "BoostUse",[2] = "x2 Damage"})
                    end
                end
            end)
        end
	end,
})

Tab9:CreateSection("Waits to use 3Timestop, 1Kayoken and 1Strongest Punch")

local skillcombo
Tab9:CreateToggle({
	Name = "Use Skill Combo",
	CurrentValue = false,
	Flag = "Use Skill Combo",
	Callback = function(Value)
        skillcombo = Value
        if Value then
            skill_combo()
        end
	end,
})
function skill_combo()
    timestop_tog:Set(false)
    strongestpunch_tog:Set(false)
    kayoken_tog:Set(false)
    spawn(function()
        while wait(1) and skillcombo do
            if services.PlayerData.Skills["Time Stop"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Time Stop"]["ToUse"] >= 3 then
                if services.PlayerData.Skills["Kayoken"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Kayoken"]["ToUse"] > 0 then
                    if services.PlayerData.Skills["Strongest Punch"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Strongest Punch"]["ToUse"] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 3})
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 4})
                        while wait(1) do
                            if services.PlayerData.Skills["Time Stop"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Time Stop"]["ToUse"] >= 0 then
                                game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 7})
                            end
                            if services.PlayerData.Skills["Time Stop"]["ToUse"] <= 0 then
                                break
                            end
                        end
                    end
                end
            end
        end
    end)
end


Tab9:CreateSection("Auto Skills")
Tab9:CreateToggle({
	Name = "Toggle all auto skills",
	CurrentValue = false,
	Flag = "Toggle all auto skills",
	Callback = function(Value)
        if Value then
            berserk_tog:Set(true)
            lightspeed_tog:Set(true)
            kayoken_tog:Set(true)
            strongestpunch_tog:Set(true)
            blackhole_tog:Set(true)
            swordmaster_tog:Set(true)
            timestop_tog:Set(true)
        else
            berserk_tog:Set(false)
            lightspeed_tog:Set(false)
            kayoken_tog:Set(false)
            strongestpunch_tog:Set(false)
            blackhole_tog:Set(false)
            swordmaster_tog:Set(false)
            timestop_tog:Set(false)
        end
	end,
})

local berserk
berserk_tog = Tab9:CreateToggle({
	Name = "Auto Use Berserk",
	CurrentValue = false,
	Flag = "Auto Use Berserk",
	Callback = function(Value)
		berserk = Value
        if Value then
            spawn(function()
                while wait(1) and berserk do
                    if services.PlayerData.Skills["Berserk"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Berserk"]["ToUse"] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 1})
                    end
                end
            end)
        end
	end,
})

local lightspeed
lightspeed_tog = Tab9:CreateToggle({
	Name = "Auto Use Light Speed",
	CurrentValue = false,
	Flag = "Auto Use Light Speed",
	Callback = function(Value)
		lightspeed = Value
        if Value then
            spawn(function()
                while wait(1) and lightspeed do
                    if services.PlayerData.Skills["Light Speed"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Light Speed"]["ToUse"] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 2})
                    end
                end
            end)
        end
	end,
})

local kayoken
kayoken_tog = Tab9:CreateToggle({
	Name = "Auto Use Kayoken",
	CurrentValue = false,
	Flag = "Auto Use Kayoken",
	Callback = function(Value)
		kayoken = Value
        if Value then
            spawn(function()
                while wait(1) and kayoken do
                    if services.PlayerData.Skills["Kayoken"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Kayoken"]["ToUse"] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 3})
                    end
                end
            end)
        end
	end,
})

local Strongestpunch
strongestpunch_tog = Tab9:CreateToggle({
	Name = "Auto Use Strongest Punch",
	CurrentValue = false,
	Flag = "Auto Use Strongest Punch",
	Callback = function(Value)
		Strongestpunch = Value
        if Value then
            spawn(function()
                while wait(1) and Strongestpunch do
                    if services.PlayerData.Skills["Strongest Punch"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Strongest Punch"]["ToUse"] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 4})
                    end
                end
            end)
        end
	end,
})

local blackhole
blackhole_tog = Tab9:CreateToggle({
	Name = "Auto Use Black Hole",
	CurrentValue = false,
	Flag = "Auto Use Black Hole",
	Callback = function(Value)
		blackhole = Value
        if Value then
            spawn(function()
                while wait(1) and blackhole do
                    if services.PlayerData.Skills["Black Hole"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Black Hole"]["ToUse"] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 5})
                    end
                end
            end)
        end
	end,
})

local swordmaster
swordmaster_tog = Tab9:CreateToggle({
	Name = "Auto Use Sword Master",
	CurrentValue = false,
	Flag = "Auto Use Sword Master",
	Callback = function(Value)
		swordmaster = Value
        if Value then
            spawn(function()
                while wait(1) and swordmaster do
                    if services.PlayerData.Skills["Sword Master"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Sword Master"]["ToUse"] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 6})
                    end
                end
            end)
        end
	end,
})

local timestop
timestop_tog = Tab9:CreateToggle({
	Name = "Auto Use Time Stop",
	CurrentValue = false,
	Flag = "Auto Use Time Stop",
	Callback = function(Value)
		timestop = Value
        if Value then
            spawn(function()
                while wait(1) and timestop do
                    if services.PlayerData.Skills["Time Stop"]["TimeLeft"] <= 0 and services.PlayerData.Skills["Time Stop"]["ToUse"] > 0 then
                        game.ReplicatedStorage.Remotes.Client:FireServer({[1] = "SkillUse",[2] = 7})
                    end
                end
            end)
        end
	end,
})

local Tab10 = Window:CreateTab("Credits", 4483362458)
Tab10:CreateLabel("GUIlib by shlexware, search up Rayfield on google")
Tab10:CreateLabel("Rest of the stuff by Perfectus#0001")
Tab10:CreateLabel("Discord Link: https://discord.com/invite/37NZHy9gW3")
Tab10:CreateButton({
	Name = "Click To Join Discord",
	Callback = function()
		local HttpService = game:GetService("HttpService") -- hmmmmmmmmmm i wonder if i made this myself
        local RequestEnabled = (syn and syn.request) or (http and http.request) or http_request
        if RequestEnabled then
            RequestEnabled({
                Url = 'http://127.0.0.1:6463/rpc?v=1',
                Method = 'POST',
                Headers = {
                    ['Content-Type'] = 'application/json',
                    Origin = 'https://discord.com'
                },
                Body = HttpService:JSONEncode({
                    cmd = 'INVITE_BROWSER',
                    nonce = HttpService:GenerateGUID(false),
                    args = {code = "37NZHy9gW3"}
                })
            })
        end
	end,
})






local vu = game:GetService("VirtualUser")
game:GetService("Players").LocalPlayer.Idled:connect(function()
   vu:Button2Down(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
   wait(1)
   vu:Button2Up(Vector2.new(0,0),workspace.CurrentCamera.CFrame)
end)


Rayfield:LoadConfiguration()
--i know my codes ass, dont harass me
