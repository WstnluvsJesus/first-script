--[[
	WARNING: Heads up! This script has not been verified by ScriptBlox. Use at your own risk!
]]
--[[
    ================================================================================
    BloxStrike - Complete Cheat Script with Auto Fire, Vector Silent Aim, 
                 3rd Person Camera & Anti-Aim (At-Targets Backwards + Pitch Down)
    Documentation / API: https://docs.projectreal.gg/
    ================================================================================
--]]

local Rayfield = loadstring(game:HttpGet("https://sirius.menu/gen2"))()

local Window = Rayfield:CreateWindow({
    Name = "BloxStrike",
    Subtitle = "Auto Fire & Vector Silent Aim",
})

local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local CoreGui = game:GetService("CoreGui")
local Lighting = game:GetService("Lighting")
local LocalPlayer = Players.LocalPlayer

-- Global Settings Integration
getgenv().ParsaIntegratedSettings = {
    -- Auto Fire Settings
    AutoFireEnabled = true,
    AutoFireDelay = 0.05,
    AutoFireAngleFOV = 15,       -- Векторный FOV (градусы)
    
    -- Silent Aim Settings
    SilentEnabled = true,
    WallCheck = true,
    Wallbang = false,
    HitPart = "Head",
    UseFOV = true,
    FOVRadius = 140,
    FOVColor = Color3.fromRGB(255, 0, 0),
    TeamCheck = true,            -- Проверка на свои команды (не стрелять по своим)

    -- Precision Chams, Bullet Tracers & Effects
    ChamsEnabled = true,
    ChamsMode = "Precision Body Chams", -- "Precision Body Chams", "Highlight Chams"
    ChamsColor = Color3.fromRGB(255, 41, 116),
    UseTeamColorChams = false,
    ChamsFillTransparency = 0.3,

    BulletTracers = true,
    TracerColor = Color3.fromRGB(255, 41, 116),
    TracerDuration = 1.2,

    NightMode = false,
    HitExplosion = true,
    
    -- Visuals & Camera Settings
    ThirdPerson = false,
    ThirdPersonDist = 12,
    HideViewModelInThirdPerson = true, -- Скрывать 1st person оружие в режиме 3 лица
    
    -- Anti-Aim Settings
    AntiAim = false,
    AntiAimPitchMode = "Down",   -- "Down", "Up", "Jitter", "Custom"
    AntiAimAngle = -89,
    AntiAimYawMode = "AtTargets",-- "AtTargets" (СПИНОЙ ИМЕННО К ВРАГУ), "Backwards", "Spinbot", "Jitter", "Sideways", "Off"
    AntiAimSpinSpeed = 15,
    AntiAimDisableOnFire = true,
    IsFiringState = false,
    
    -- ESP Settings
    ESPEnabled = false,
    ESPBoxes = false,
    ESPSkeletons = false,
    ESPNames = false,
    ESPHealth = false,
    ESPDistance = false,
    ESPMaxDistance = 2500,
    BoxColor = Color3.fromRGB(0, 186, 186),
    OutlineColor = Color3.fromRGB(15, 15, 15),
    NameColor = Color3.fromRGB(255, 255, 255),
    DistanceColor = Color3.fromRGB(179, 179, 179)
}

getgenv().ParsaTeamColors = {
    ["Terrorists"] = Color3.fromRGB(204, 170, 80),
    ["Counter-Terrorists"] = Color3.fromRGB(100, 149, 200)
}

getgenv().ParsaSilentTarget = nil
local Settings = getgenv().ParsaIntegratedSettings

-- ==========================================
-- Векторная математика (Vector Math Functions)
-- ==========================================
local VectorMath = {}

function VectorMath.getDirectionUnit(origin: Vector3, target: Vector3): Vector3
    local delta = target - origin
    if delta.Magnitude < 0.0001 then
        return Vector3.new(0, 0, 1)
    end
    return delta.Unit
end

function VectorMath.getAngleDegrees(lookVector: Vector3, targetDirUnit: Vector3): number
    local dot = math.clamp(lookVector:Dot(targetDirUnit), -1, 1)
    return math.deg(math.acos(dot))
end

function VectorMath.isInVectorFOV(origin: Vector3, lookVector: Vector3, targetPos: Vector3, maxAngleDeg: number): (boolean, number)
    local dirUnit = VectorMath.getDirectionUnit(origin, targetPos)
    local angleDeg = VectorMath.getAngleDegrees(lookVector, dirUnit)
    return (angleDeg <= maxAngleDeg), angleDeg
end

-- ==========================================
-- Логика Определения Противоположной Команды
-- ==========================================
local function getNormalizedSide(player: Player): string?
    if not player then return nil end

    local ok, teamObj = pcall(function() return player.Team end)
    if ok and teamObj and teamObj.Name then
        local name = teamObj.Name:lower()
        if name:find("counter") or name == "ct" or name:find("blue") or name:find("guard") or name:find("cop") then
            return "CT"
        end
        if name:find("terror") or name == "t" or name:find("red") or name:find("robber") or name:find("inmate") then
            return "T"
        end
    end

    local okC, teamColor = pcall(function() return player.TeamColor end)
    if okC and teamColor then
        local cName = teamColor.Name:lower()
        if cName:find("blue") or cName:find("navy") or cName:find("cyan") then return "CT" end
        if cName:find("red") or cName:find("orange") or cName:find("yellow") or cName:find("brown") then return "T" end
    end

    local char = player.Character
    if char and char.Parent then
        local pName = char.Parent.Name:lower()
        if pName:find("counter") or pName == "ct" then return "CT" end
        if pName:find("terror") or pName == "t" then return "T" end
    end

    local attr = player:GetAttribute("Team") or player:GetAttribute("Side") or (char and (char:GetAttribute("Team") or char:GetAttribute("Side")))
    if attr ~= nil then
        local s = tostring(attr):lower()
        if s:find("counter") or s == "ct" or s == "2" then return "CT" end
        if s:find("terror") or s == "t" or s == "1" then return "T" end
    end

    local val = (char and (char:FindFirstChild("Team") or char:FindFirstChild("Side"))) or player:FindFirstChild("Team") or player:FindFirstChild("Side")
    if val and val:IsA("ValueBase") then
        local s = tostring(val.Value):lower()
        if s:find("counter") or s == "ct" or s == "2" then return "CT" end
        if s:find("terror") or s == "t" or s == "1" then return "T" end
    end

    local leaderstats = player:FindFirstChild("leaderstats")
    if leaderstats then
        local stat = leaderstats:FindFirstChild("Team") or leaderstats:FindFirstChild("Side")
        if stat and stat:IsA("ValueBase") then
            local s = tostring(stat.Value):lower()
            if s:find("counter") or s == "ct" or s == "2" then return "CT" end
            if s:find("terror") or s == "t" or s == "1" then return "T" end
        end
    end

    return nil
end

local function isOppositeTeam(myPlayer: Player, targetPlayer: Player): boolean
    if not myPlayer or not targetPlayer then return false end
    if myPlayer == targetPlayer then return false end

    local mySide = getNormalizedSide(myPlayer)
    local targetSide = getNormalizedSide(targetPlayer)

    if mySide and targetSide then
        return (mySide ~= targetSide)
    end

    local ok1, team1 = pcall(function() return myPlayer.Team end)
    local ok2, team2 = pcall(function() return targetPlayer.Team end)
    if ok1 and ok2 and team1 ~= nil and team2 ~= nil then
        return team1 ~= team2
    end

    local okC1, color1 = pcall(function() return myPlayer.TeamColor end)
    local okC2, color2 = pcall(function() return targetPlayer.TeamColor end)
    if okC1 and okC2 and color1 ~= nil and color2 ~= nil then
        return color1 ~= color2
    end

    return true
end

getgenv().ParsaGetTeam = getNormalizedSide
getgenv().ParsaIsOpposite = isOppositeTeam

local function get_player_team(player: Player): string?
    local side = getNormalizedSide(player)
    if side == "CT" then return "Counter-Terrorists" end
    if side == "T" then return "Terrorists" end
    return nil
end

local function getGuiContainer()
    local container
    pcall(function()
        if typeof(gethui) == "function" then
            container = gethui()
        elseif CoreGui then
            container = CoreGui
        end
    end)
    if not container then
        pcall(function()
            container = LocalPlayer:WaitForChild("PlayerGui")
        end)
    end
    return container or Workspace
end

-- ==========================================
-- Спавн 3D Neon Bullet Tracers
-- ==========================================
local function spawnBulletTracer(fromPos: Vector3, toPos: Vector3)
    if not Settings.BulletTracers then return end
    local distance = (toPos - fromPos).Magnitude
    if distance < 0.2 then return end

    local beamPart = Instance.new("Part")
    beamPart.Name = "ParsaBulletTracer"
    beamPart.Anchored = true
    beamPart.CanCollide = false
    beamPart.CastShadow = false
    beamPart.Material = Enum.Material.Neon
    beamPart.Color = Settings.TracerColor or Color3.fromRGB(255, 41, 116)
    beamPart.Transparency = 0.15

    beamPart.Size = Vector3.new(0.08, 0.08, distance)
    beamPart.CFrame = CFrame.new(fromPos, toPos) * CFrame.new(0, 0, -distance / 2)
    beamPart.Parent = Workspace

    local tweenInfo = TweenInfo.new(Settings.TracerDuration or 1.2, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
    local tween = TweenService:Create(beamPart, tweenInfo, { Transparency = 1 })
    tween:Play()
    tween.Completed:Connect(function() beamPart:Destroy() end)

    task.delay((Settings.TracerDuration or 1.2) + 0.2, function()
        if beamPart and beamPart.Parent then beamPart:Destroy() end
    end)
end

-- ==========================================
-- Спавн Эффекта Взрыва (Hit / Explosion Effect)
-- ==========================================
local function spawnHitExplosion(pos: Vector3)
    if not Settings.HitExplosion or not pos then return end
    pcall(function()
        local exp = Instance.new("Explosion")
        exp.BlastPressure = 0
        exp.BlastRadius = 0
        exp.DestroyColliderRadius = 0
        exp.ExplosionType = Enum.ExplosionType.NoCulling
        exp.Position = pos
        exp.Parent = Workspace

        local shockwave = Instance.new("Part")
        shockwave.Name = "ParsaExplosionShockwave"
        shockwave.Shape = Enum.PartType.Ball
        shockwave.Material = Enum.Material.Neon
        shockwave.Color = Color3.fromRGB(255, 120, 30)
        shockwave.Transparency = 0.2
        shockwave.Anchored = true
        shockwave.CanCollide = false
        shockwave.Size = Vector3.new(0.5, 0.5, 0.5)
        shockwave.CFrame = CFrame.new(pos)
        shockwave.Parent = Workspace

        local shockwaveTween = TweenService:Create(shockwave, TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out), {
            Size = Vector3.new(7, 7, 7),
            Transparency = 1
        })
        shockwaveTween:Play()
        shockwaveTween.Completed:Connect(function() shockwave:Destroy() end)
    end)
end

-- ==========================================
-- Night Bot (Атмосфера Ночного Режима)
-- ==========================================
local function applyNightState()
    if Settings.NightMode then
        Lighting.ClockTime = 0
        Lighting.Brightness = 2
        Lighting.OutdoorAmbient = Color3.fromRGB(70, 70, 90)
        Lighting.Ambient = Color3.fromRGB(60, 60, 70)
        Lighting.GlobalShadows = true
        Lighting.ExposureCompensation = 0.5
    else
        Lighting.ClockTime = 14
        Lighting.Brightness = 3
        Lighting.OutdoorAmbient = Color3.fromRGB(127, 127, 127)
        Lighting.Ambient = Color3.fromRGB(0, 0, 0)
        Lighting.ExposureCompensation = 0
    end
end

-- ==========================================
-- Высокоточные Чамсы Скелета и Тела
-- ==========================================
local partChamsCache = {}
local highlightChamsCache = {}

local function isVisibleBodyPart(part)
    if not part:IsA("BasePart") then return false end
    local name = part.Name:lower()
    if name == "humanoidrootpart" or name == "root" or name == "hitbox" or name == "collision" then
        return false
    end
    if part.Transparency >= 0.99 then return false end
    return true
end

local function updateChams()
    if not Settings.ChamsEnabled then
        for player, partMap in pairs(partChamsCache) do
            for part, adornment in pairs(partMap) do
                if adornment and typeof(adornment) == "Instance" then adornment:Destroy() end
            end
        end
        partChamsCache = {}
        for player, highlight in pairs(highlightChamsCache) do
            if highlight and typeof(highlight) == "Instance" then highlight:Destroy() end
        end
        highlightChamsCache = {}
        return
    end

    for player, partMap in pairs(partChamsCache) do
        if not player or not player.Parent then
            for part, adornment in pairs(partMap) do
                if adornment then adornment:Destroy() end
            end
            partChamsCache[player] = nil
        end
    end
    for player, highlight in pairs(highlightChamsCache) do
        if not player or not player.Parent then
            if highlight then highlight:Destroy() end
            highlightChamsCache[player] = nil
        end
    end

    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        local char = player.Character
        local humanoid = char and char:FindFirstChildOfClass("Humanoid")
        local isDead = char and (char:GetAttribute("Dead") == true or (humanoid and humanoid.Health <= 0))
        local allowed = char and not isDead and (not Settings.TeamCheck or isOppositeTeam(LocalPlayer, player))

        if allowed then
            local chamColor = Settings.ChamsColor
            if Settings.UseTeamColorChams then
                local side = getNormalizedSide(player)
                if side == "T" then chamColor = Color3.fromRGB(240, 160, 40)
                elseif side == "CT" then chamColor = Color3.fromRGB(50, 140, 240) end
            end

            if Settings.ChamsMode == "Precision Body Chams" then
                if highlightChamsCache[player] then
                    highlightChamsCache[player]:Destroy()
                    highlightChamsCache[player] = nil
                end

                if not partChamsCache[player] then partChamsCache[player] = {} end
                local partMap = partChamsCache[player]

                for _, child in ipairs(char:GetChildren()) do
                    if isVisibleBodyPart(child) then
                        local adornment = partMap[child]
                        if not adornment or not adornment.Parent or adornment.Parent ~= child then
                            if adornment then adornment:Destroy() end
                            adornment = Instance.new("BoxHandleAdornment")
                            adornment.Name = "Cham_" .. child.Name
                            adornment.AlwaysOnTop = true
                            adornment.ZIndex = 10
                            adornment.Parent = child
                            partMap[child] = adornment
                        end

                        adornment.Adornee = child
                        adornment.Size = child.Size
                        adornment.Color3 = chamColor
                        adornment.Transparency = Settings.ChamsFillTransparency or 0.3
                        adornment.Visible = true
                    end
                end

                for part, adornment in pairs(partMap) do
                    if not part or not part.Parent or part.Parent ~= char then
                        if adornment then adornment:Destroy() end
                        partMap[part] = nil
                    end
                end

            elseif Settings.ChamsMode == "Highlight Chams" then
                if partChamsCache[player] then
                    for part, adornment in pairs(partChamsCache[player]) do
                        if adornment then adornment:Destroy() end
                    end
                    partChamsCache[player] = nil
                end

                local highlight = highlightChamsCache[player]
                if not highlight or not highlight.Parent then
                    highlight = Instance.new("Highlight")
                    highlight.Name = "FatalityChams_" .. player.Name
                    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                    highlight.Parent = char
                    highlightChamsCache[player] = highlight
                end

                if highlight.Adornee ~= char then highlight.Adornee = char end
                highlight.Enabled = true
                highlight.FillColor = chamColor
                highlight.OutlineColor = chamColor
                highlight.FillTransparency = Settings.ChamsFillTransparency or 0.4
                highlight.OutlineTransparency = 0
            end

        else
            if partChamsCache[player] then
                for part, adornment in pairs(partMap) do
                    if adornment then adornment:Destroy() end
                end
                partChamsCache[player] = nil
            end
            if highlightChamsCache[player] then
                highlightChamsCache[player]:Destroy()
                highlightChamsCache[player] = nil
            end
        end
    end
end

-- ==========================================
-- Поиск Ближайшего Противника (Противоположная Команда)
-- ==========================================
local function getClosestEnemyPart(): BasePart?
    local Camera = Workspace.CurrentCamera
    local char = LocalPlayer.Character
    if not char then return nil end
    local root = char:FindFirstChild("HumanoidRootPart")
    if not root then return nil end

    local myPos = root.Position
    local closestDist = math.huge
    local closestPart = nil

    for _, player in ipairs(Players:GetPlayers()) do
        if player == LocalPlayer then continue end
        local pChar = player.Character
        if not pChar then continue end

        local humanoid = pChar:FindFirstChildOfClass("Humanoid")
        local isDead = pChar:GetAttribute("Dead") == true or (humanoid and humanoid.Health <= 0)
        local isInvincible = pChar:GetAttribute("Invincible") == true
        if isDead or isInvincible then continue end
        if pChar.Parent and pChar.Parent.Name == "Debris" then continue end

        -- Атакуем ТОЛЬКО противоположную команду!
        if not isOppositeTeam(LocalPlayer, player) then continue end

        local ePart = pChar:FindFirstChild("HumanoidRootPart") or pChar:FindFirstChild("Head") or pChar.PrimaryPart
        if ePart then
            local dist = (ePart.Position - myPos).Magnitude
            if dist < closestDist then
                closestDist = dist
                closestPart = ePart
            end
        end
    end

    return closestPart
end

-- ==========================================
-- Интерфейс Пользователя (UI Tabs)
-- ==========================================
local CombatTab = Window:CreateTab({ Name = "Main", Icon = 10734950309 })
local VisualsTab = Window:CreateTab({ Name = "Visuals & Camera", Icon = 10723345474 })

-- Combat Controls
CombatTab:CreateToggle({
    Name = "Auto Fire",
    CurrentValue = Settings.AutoFireEnabled,
    Callback = function(Value) Settings.AutoFireEnabled = Value end,
})

CombatTab:CreateSlider({
    Name = "Vector FOV Angle",
    Range = {1, 60},
    Increment = 1,
    Suffix = "°",
    CurrentValue = Settings.AutoFireAngleFOV,
    Callback = function(Value) Settings.AutoFireAngleFOV = Value end,
})

CombatTab:CreateToggle({
    Name = "Silent Aim",
    CurrentValue = Settings.SilentEnabled,
    Callback = function(Value) Settings.SilentEnabled = Value end,
})

CombatTab:CreateToggle({
    Name = "Wallbang",
    CurrentValue = Settings.Wallbang,
    Callback = function(Value) Settings.Wallbang = Value end,
})

CombatTab:CreateToggle({
    Name = "Team Check (Атаковать только врагов)",
    CurrentValue = Settings.TeamCheck,
    Callback = function(Value) Settings.TeamCheck = Value end,
})

CombatTab:CreateDropdown({
    Name = "Target Hitpart",
    Options = {"Head", "UpperTorso", "LowerTorso"},
    CurrentOption = {Settings.HitPart},
    MultipleOptions = false,
    Callback = function(Options) 
        if type(Options) == "table" then
            Settings.HitPart = Options[1]
        else
            Settings.HitPart = Options
        end
    end,
})

CombatTab:CreateToggle({
    Name = "Show Screen FOV",
    CurrentValue = Settings.UseFOV,
    Callback = function(Value) Settings.UseFOV = Value end,
})

CombatTab:CreateSlider({
    Name = "FOV Radius",
    Range = {10, 500},
    Increment = 5,
    Suffix = "px",
    CurrentValue = Settings.FOVRadius,
    Callback = function(Value) Settings.FOVRadius = Value end,
})

-- Visuals & Camera Controls
VisualsTab:CreateToggle({
    Name = "Chams Master Enable (Чамсы)",
    CurrentValue = Settings.ChamsEnabled,
    Callback = function(Value) Settings.ChamsEnabled = Value end,
})

VisualsTab:CreateDropdown({
    Name = "Chams Style Mode",
    Options = {"Precision Body Chams", "Highlight Chams"},
    CurrentOption = {Settings.ChamsMode},
    MultipleOptions = false,
    Callback = function(Options)
        Settings.ChamsMode = type(Options) == "table" and Options[1] or Options
    end,
})

VisualsTab:CreateToggle({
    Name = "3D Neon Bullet Tracers (Трейсеры пуль)",
    CurrentValue = Settings.BulletTracers,
    Callback = function(Value) Settings.BulletTracers = Value end,
})

VisualsTab:CreateSlider({
    Name = "Tracer Fade Duration",
    Range = {0.5, 4},
    Increment = 0.1,
    Suffix = "s",
    CurrentValue = Settings.TracerDuration,
    Callback = function(Value) Settings.TracerDuration = Value end,
})

VisualsTab:CreateToggle({
    Name = "Night Mode / Night Bot (Ночной режим)",
    CurrentValue = Settings.NightMode,
    Callback = function(Value) Settings.NightMode = Value end,
})

VisualsTab:CreateToggle({
    Name = "Hit Explosion Effect (Эффект взрыва)",
    CurrentValue = Settings.HitExplosion,
    Callback = function(Value) Settings.HitExplosion = Value end,
})

VisualsTab:CreateToggle({
    Name = "3rd Person View (Вид от 3 лица)",
    CurrentValue = Settings.ThirdPerson,
    Callback = function(Value) Settings.ThirdPerson = Value end,
})

VisualsTab:CreateSlider({
    Name = "3rd Person Distance",
    Range = {5, 30},
    Increment = 1,
    Suffix = " studs",
    CurrentValue = Settings.ThirdPersonDist,
    Callback = function(Value) Settings.ThirdPersonDist = Value end,
})

VisualsTab:CreateToggle({
    Name = "Hide 1st Person ViewModel (Вырубать оружие в 3-м лице)",
    CurrentValue = Settings.HideViewModelInThirdPerson,
    Callback = function(Value) Settings.HideViewModelInThirdPerson = Value end,
})

VisualsTab:CreateToggle({
    Name = "Anti-Aim (Включить Анти-Аим)",
    CurrentValue = Settings.AntiAim,
    Callback = function(Value) Settings.AntiAim = Value end,
})

VisualsTab:CreateDropdown({
    Name = "Anti-Aim Pitch Mode (Режим наклона)",
    Options = {"Down", "Up", "Jitter", "Custom"},
    CurrentOption = {Settings.AntiAimPitchMode},
    MultipleOptions = false,
    Callback = function(Options)
        Settings.AntiAimPitchMode = type(Options) == "table" and Options[1] or Options
    end,
})

VisualsTab:CreateSlider({
    Name = "Anti-Aim Custom Pitch Angle",
    Range = {-89, 89},
    Increment = 1,
    Suffix = "°",
    CurrentValue = Settings.AntiAimAngle,
    Callback = function(Value) Settings.AntiAimAngle = Value end,
})

VisualsTab:CreateDropdown({
    Name = "Anti-Aim Yaw Mode (Режим вращения)",
    Options = {"AtTargets", "Backwards", "Spinbot", "Jitter", "Sideways", "Off"},
    CurrentOption = {Settings.AntiAimYawMode},
    MultipleOptions = false,
    Callback = function(Options)
        Settings.AntiAimYawMode = type(Options) == "table" and Options[1] or Options
    end,
})

VisualsTab:CreateSlider({
    Name = "Spinbot Speed",
    Range = {1, 50},
    Increment = 1,
    Suffix = " deg/f",
    CurrentValue = Settings.AntiAimSpinSpeed,
    Callback = function(Value) Settings.AntiAimSpinSpeed = Value end,
})

VisualsTab:CreateToggle({
    Name = "Disable Anti-Aim On Fire (Пауза AA при стрельбе)",
    CurrentValue = Settings.AntiAimDisableOnFire,
    Callback = function(Value) Settings.AntiAimDisableOnFire = Value end,
})

VisualsTab:CreateToggle({
    Name = "ESP",
    CurrentValue = Settings.ESPEnabled,
    Callback = function(Value) Settings.ESPEnabled = Value end,
})

VisualsTab:CreateToggle({
    Name = "Draw Boxes",
    CurrentValue = Settings.ESPBoxes,
    Callback = function(Value) Settings.ESPBoxes = Value end,
})

VisualsTab:CreateToggle({
    Name = "Draw Skeletons",
    CurrentValue = Settings.ESPSkeletons,
    Callback = function(Value) Settings.ESPSkeletons = Value end,
})

VisualsTab:CreateToggle({
    Name = "Display Names",
    CurrentValue = Settings.ESPNames,
    Callback = function(Value) Settings.ESPNames = Value end,
})

VisualsTab:CreateToggle({
    Name = "Display Health",
    CurrentValue = Settings.ESPHealth,
    Callback = function(Value) Settings.ESPHealth = Value end,
})

VisualsTab:CreateToggle({
    Name = "Display Distance",
    CurrentValue = Settings.ESPDistance,
    Callback = function(Value) Settings.ESPDistance = Value end,
})

-- ==========================================
-- ESP Системы (Drawing API)
-- ==========================================
local Camera = Workspace.CurrentCamera
getgenv().ParsaEspInstances = getgenv().ParsaEspInstances or {}
local espInstances = getgenv().ParsaEspInstances

local BONE_MAP_R15 = {
    {"Head", "UpperTorso"}, {"UpperTorso", "LowerTorso"},
    {"UpperTorso", "LeftUpperArm"}, {"LeftUpperArm", "LeftLowerArm"}, {"LeftLowerArm", "LeftHand"},
    {"UpperTorso", "RightUpperArm"}, {"RightUpperArm", "RightLowerArm"}, {"RightLowerArm", "RightHand"},
    {"LowerTorso", "LeftUpperLeg"}, {"LeftUpperLeg", "LeftLowerLeg"}, {"LeftLowerLeg", "LeftFoot"},
    {"LowerTorso", "RightUpperLeg"}, {"RightUpperLeg", "RightLowerLeg"}, {"RightLowerLeg", "RightFoot"}
}

local BONE_MAP_R6 = {
    {"Head", "Torso"}, {"Torso", "Left Arm"}, {"Torso", "Right Arm"},
    {"Torso", "Left Leg"}, {"Torso", "Right Leg"}
}

local function createESP(player)
    if espInstances[player] then return end
    local objects = {
        BoxOutline = Drawing.new("Square"),
        Box = Drawing.new("Square"),
        Name = Drawing.new("Text"),
        Distance = Drawing.new("Text"),
        HealthOutline = Drawing.new("Square"),
        Health = Drawing.new("Square"),
        Bones = {}
    }
    objects.BoxOutline.Thickness = 2
    objects.BoxOutline.Filled = false
    objects.BoxOutline.Transparency = 0.5
    objects.BoxOutline.Color = Settings.OutlineColor
    
    objects.Box.Thickness = 1
    objects.Box.Filled = false
    objects.Box.Transparency = 1

    objects.Name.Center = true
    objects.Name.Outline = true
    objects.Name.Font = 1
    objects.Name.Size = 13
    objects.Name.Color = Settings.NameColor

    objects.Distance.Center = true
    objects.Distance.Outline = true
    objects.Distance.Font = 1
    objects.Distance.Size = 12
    objects.Distance.Color = Settings.DistanceColor

    objects.HealthOutline.Thickness = 1
    objects.HealthOutline.Filled = true
    objects.HealthOutline.Transparency = 0.5
    objects.HealthOutline.Color = Settings.OutlineColor
    objects.Health.Filled = true
    objects.Health.Transparency = 1

    for i = 1, 15 do
        local boneLine = Drawing.new("Line")
        boneLine.Thickness = 1
        boneLine.Transparency = 1
        boneLine.Visible = false
        table.insert(objects.Bones, boneLine)
    end
    espInstances[player] = objects
end

local function removeESP(player)
    local objects = espInstances[player]
    if objects then
        for _, obj in pairs(objects) do
            if type(obj) == "table" then
                for _, bone in pairs(obj) do bone:Remove() end
            else
                obj:Remove()
            end
        end
        espInstances[player] = nil
    end
end
getgenv().ParsaRemoveEsp = removeESP

-- ==========================================
-- Логика Поиска Цели и Векторных Проверок
-- ==========================================
local FOVCircle = Drawing.new("Circle")
FOVCircle.Filled = false
FOVCircle.Thickness = 1
FOVCircle.NumSides = 64
FOVCircle.Color = Settings.FOVColor

local rayParams = RaycastParams.new()
rayParams.FilterType = Enum.RaycastFilterType.Exclude
rayParams.IgnoreWater = true

local function isVisible(target)
    if Settings.Wallbang then return true end
    if not target or not target.Parent or not Camera then return false end
    
    rayParams.FilterDescendantsInstances = LocalPlayer.Character and { LocalPlayer.Character } or {}
    local origin = Camera.CFrame.Position
    local result = Workspace:Raycast(origin, target.Position - origin, rayParams)
    if not result then return true end
    local model = result.Instance and result.Instance:FindFirstAncestorOfClass("Model")
    return model and Players:GetPlayerFromCharacter(model) ~= nil
end

local function FindSilentTarget()
    Camera = Workspace.CurrentCamera
    if not Camera or not LocalPlayer.Character then 
        getgenv().ParsaSilentTarget = nil 
        return nil
    end
    
    local camPos = Camera.CFrame.Position
    local lookVec = Camera.CFrame.LookVector
    local screenCenter = Camera.ViewportSize / 2
    local closestDist = math.huge
    local closestPart = nil

    for _, player in next, Players:GetPlayers() do
        if player == LocalPlayer then continue end
        local char = player.Character
        if not char or char:GetAttribute("Dead") or char:GetAttribute("Invincible") then continue end
        if char.Parent and char.Parent.Name == "Debris" then continue end

        -- АТАКА ТОЛЬКО ПРОТИВОПОЛОЖНОЙ КОМАНДЫ
        if Settings.TeamCheck and not isOppositeTeam(LocalPlayer, player) then continue end

        local targetPart = char:FindFirstChild(Settings.HitPart) or char:FindFirstChild("Head") or char.PrimaryPart
        if not targetPart then continue end

        -- Векторная проверка угла (Vector3 & Unit Vector & Dot Product)
        local inVectorFOV, angleDeg = VectorMath.isInVectorFOV(camPos, lookVec, targetPart.Position, Settings.AutoFireAngleFOV)
        
        local screenPos, onScreen = Camera:WorldToViewportPoint(targetPart.Position)
        if onScreen then
            local distFromCenter = (Vector2.new(screenPos.X, screenPos.Y) - screenCenter).Magnitude
            local maxRadius = Settings.UseFOV and Settings.FOVRadius or math.huge
            
            if inVectorFOV and distFromCenter <= maxRadius and distFromCenter < closestDist then
                if Settings.Wallbang or isVisible(targetPart) then
                    closestDist = distFromCenter
                    closestPart = targetPart
                end
            end
        end
    end
    getgenv().ParsaSilentTarget = closestPart
    return closestPart
end

-- ==========================================
-- Исполнение Выстрелов (Real API)
-- ==========================================
local lastFireTime = 0
local function executeAutoFire()
    local now = os.clock()
    if now - lastFireTime < Settings.AutoFireDelay then return end
    lastFireTime = now

    if typeof(isrbxactive) == "function" and isrbxactive() then
        if typeof(mouse1click) == "function" then
            mouse1click()
        elseif typeof(mouse1press) == "function" and typeof(mouse1release) == "function" then
            mouse1press()
            task.delay(0.02, mouse1release)
        end
    else
        pcall(function()
            local vim = game:GetService("VirtualInputManager")
            vim:SendMouseButtonEvent(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2, 0, true, game, 0)
            task.delay(0.02, function()
                vim:SendMouseButtonEvent(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y / 2, 0, false, game, 0)
            end)
        end)
    end
end

-- ==========================================
-- Anti-Aim Pitch & Yaw (СПИНОЙ ИМЕННО К ВРАГУ)
-- ==========================================
local originalNeckC0 = nil
local originalWaistC0 = nil
local originalRootC0 = nil
local lastCharAA = nil
local spinAngle = 0

local function getJoints()
    local char = LocalPlayer.Character
    if not char then return nil, nil, nil, nil end

    local head = char:FindFirstChild("Head")
    local upperTorso = char:FindFirstChild("UpperTorso") or char:FindFirstChild("Torso")
    local lowerTorso = char:FindFirstChild("LowerTorso")
    local rootPart = char:FindFirstChild("HumanoidRootPart")
    local humanoid = char:FindFirstChildOfClass("Humanoid")

    local neck = (head and head:FindFirstChild("Neck")) or (upperTorso and upperTorso:FindFirstChild("Neck"))
    local waist = (upperTorso and upperTorso:FindFirstChild("Waist")) or (lowerTorso and lowerTorso:FindFirstChild("Waist"))
    local rootJoint = rootPart and rootPart:FindFirstChild("RootJoint")

    return neck, waist, rootJoint, humanoid
end

local function updateAntiAimState()
    local char = LocalPlayer.Character
    if not char then return end

    if char ~= lastCharAA then
        lastCharAA = char
        originalNeckC0 = nil
        originalWaistC0 = nil
        originalRootC0 = nil
    end

    local neck, waist, rootJoint, humanoid = getJoints()

    if neck and not originalNeckC0 then originalNeckC0 = neck.C0 end
    if waist and not originalWaistC0 then originalWaistC0 = waist.C0 end
    if rootJoint and not originalRootC0 then originalRootC0 = rootJoint.C0 end

    if not Settings.AntiAim then
        if neck and originalNeckC0 then neck.C0 = originalNeckC0 end
        if waist and originalWaistC0 then waist.C0 = originalWaistC0 end
        if rootJoint and originalRootC0 then rootJoint.C0 = originalRootC0 end
        if humanoid then humanoid.AutoRotate = true end
        return
    end

    if humanoid then humanoid.AutoRotate = false end

    -- Если включена пауза при стрельбе и идет выстрел, выпрямляем персонажа
    if Settings.AntiAimDisableOnFire and Settings.IsFiringState then
        if neck and originalNeckC0 then neck.C0 = originalNeckC0 end
        if waist and originalWaistC0 then waist.C0 = originalWaistC0 end
        if rootJoint and originalRootC0 then rootJoint.C0 = originalRootC0 end
        return
    end

    -- Находим противника из противоположной команды
    local enemyPart = getgenv().ParsaSilentTarget or getClosestEnemyPart()
    local rootPart = char:FindFirstChild("HumanoidRootPart")

    -- Динамически разворачиваем персонажа СПИНОЙ ИМЕННО К ПОЗИЦИИ ВРАГА!
    if rootPart then
        local myPos = rootPart.Position
        local enemyPos = enemyPart and enemyPart.Position

        if enemyPos then
            local delta = Vector3.new(enemyPos.X - myPos.X, 0, enemyPos.Z - myPos.Z)
            if delta.Magnitude > 0.001 then
                -- Лицом от врага -> СПИНА НАПРАВЛЕНА ПРЯМО НА ВРАГА
                rootPart.CFrame = CFrame.lookAt(myPos, myPos - delta.Unit)
            end
        elseif Camera then
            local lookDir = Camera.CFrame.LookVector
            local flatLook = Vector3.new(lookDir.X, 0, lookDir.Z)
            if flatLook.Magnitude > 0.001 then
                rootPart.CFrame = CFrame.lookAt(myPos, myPos - flatLook.Unit)
            end
        end
    end

    -- Расчет Pitch
    local pitchRad = 0
    local pitchMode = Settings.AntiAimPitchMode or "Down"
    if pitchMode == "Down" then
        pitchRad = math.rad(-89)
    elseif pitchMode == "Up" then
        pitchRad = math.rad(89)
    elseif pitchMode == "Jitter" then
        pitchRad = math.rad(math.random() > 0.5 and -89 or 89)
    elseif pitchMode == "Custom" then
        pitchRad = math.rad(Settings.AntiAimAngle or -89)
    end

    -- Расчет Yaw
    local yawRad = 0
    local yawMode = Settings.AntiAimYawMode or "AtTargets"
    if yawMode == "Spinbot" then
        spinAngle = (spinAngle + (Settings.AntiAimSpinSpeed or 15)) % 360
        yawRad = math.rad(spinAngle)
    elseif yawMode == "Jitter" then
        yawRad = math.rad(math.random() > 0.5 and 90 or -90)
    elseif yawMode == "Sideways" then
        yawRad = math.rad(90)
    elseif yawMode == "Backwards" then
        yawRad = math.pi
    end

    -- Применяем CFrame к суставам
    if neck and originalNeckC0 then
        neck.C0 = originalNeckC0 * CFrame.Angles(pitchRad, yawRad, 0)
    end

    if waist and originalWaistC0 then
        waist.C0 = originalWaistC0 * CFrame.Angles(pitchRad * 0.4, 0, 0)
    end

    if rootJoint and originalRootC0 and yawMode == "Spinbot" then
        rootJoint.C0 = originalRootC0 * CFrame.Angles(0, 0, yawRad)
    elseif rootJoint and originalRootC0 then
        rootJoint.C0 = originalRootC0
    end
end

-- ==========================================
-- Скрытие 1st Person Viewmodel при 3rd Person
-- ==========================================
local function setViewModelVisibility(visible: boolean)
    Camera = Workspace.CurrentCamera
    local targetAlpha = visible and 0 or 1

    if Camera then
        for _, child in ipairs(Camera:GetChildren()) do
            if child:IsA("Model") or child:IsA("Folder") or child:IsA("BasePart") or child.Name:lower():find("view") or child.Name:lower():find("arm") or child.Name:lower():find("gun") then
                for _, part in ipairs(child:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.LocalTransparencyModifier = targetAlpha
                        part.Transparency = targetAlpha
                    elseif part:IsA("Decal") or part:IsA("Texture") then
                        part.Transparency = targetAlpha
                    end
                end
                if child:IsA("BasePart") then
                    child.LocalTransparencyModifier = targetAlpha
                    child.Transparency = targetAlpha
                end
            end
        end
    end

    for _, child in ipairs(Workspace:GetChildren()) do
        local n = child.Name:lower()
        if (n:find("viewmodel") or n == "vmodel" or n:find("fps")) and (child:IsA("Model") or child:IsA("Folder") or child:IsA("BasePart")) then
            for _, part in ipairs(child:GetDescendants()) do
                if part:IsA("BasePart") then
                    part.LocalTransparencyModifier = targetAlpha
                    part.Transparency = targetAlpha
                elseif part:IsA("Decal") or part:IsA("Texture") then
                    part.Transparency = targetAlpha
                end
            end
            if child:IsA("BasePart") then
                child.LocalTransparencyModifier = targetAlpha
                child.Transparency = targetAlpha
            end
        end
    end

    local char = LocalPlayer.Character
    if char then
        for _, child in ipairs(char:GetChildren()) do
            local n = child.Name:lower()
            if (n:find("viewmodel") or n == "vmodel" or n:find("fps")) and (child:IsA("Model") or child:IsA("Folder")) then
                for _, part in ipairs(child:GetDescendants()) do
                    if part:IsA("BasePart") then
                        part.LocalTransparencyModifier = targetAlpha
                        part.Transparency = targetAlpha
                    elseif part:IsA("Decal") or part:IsA("Texture") then
                        part.Transparency = targetAlpha
                    end
                end
            end
        end
    end
end

-- ==========================================
-- 3rd Person View Logic
-- ==========================================
local lastThirdPersonState = false
local originalMinZoom = 0.5
local originalMaxZoom = 128

local function updateThirdPersonState()
    if not LocalPlayer then return end
    local char = LocalPlayer.Character
    Camera = Workspace.CurrentCamera

    if Settings.ThirdPerson then
        local dist = Settings.ThirdPersonDist or 12
        LocalPlayer.CameraMode = Enum.CameraMode.Classic
        LocalPlayer.CameraMinZoomDistance = dist
        LocalPlayer.CameraMaxZoomDistance = dist

        if char and Camera then
            local head = char:FindFirstChild("Head") or char:FindFirstChild("HumanoidRootPart")
            if head then
                for _, descendant in ipairs(char:GetDescendants()) do
                    if descendant:IsA("BasePart") then
                        descendant.LocalTransparencyModifier = 0
                    end
                end

                if Settings.HideViewModelInThirdPerson then
                    setViewModelVisibility(false)
                end

                local focusPos = head.Position + Vector3.new(0, 1.5, 0)
                local lookDir = Camera.CFrame.LookVector
                local targetCamPos = focusPos - (lookDir * dist)

                local tpRayParams = RaycastParams.new()
                tpRayParams.FilterType = Enum.RaycastFilterType.Exclude
                tpRayParams.FilterDescendantsInstances = { char }

                local hitResult = Workspace:Raycast(focusPos, -lookDir * dist, tpRayParams)
                if hitResult then
                    targetCamPos = hitResult.Position + (lookDir * 0.5)
                end

                Camera.CFrame = CFrame.new(targetCamPos, focusPos)
            end
        end
        lastThirdPersonState = true
    else
        if lastThirdPersonState then
            LocalPlayer.CameraMinZoomDistance = originalMinZoom
            LocalPlayer.CameraMaxZoomDistance = originalMaxZoom
            setViewModelVisibility(true)
            lastThirdPersonState = false
        end
    end
end

-- ==========================================
-- Главный Цикл Рендера (RenderStepped Loop)
-- ==========================================
RunService.RenderStepped:Connect(function()
    Camera = Workspace.CurrentCamera
    if not Camera then return end
    
    -- Отрисовка FOV
    FOVCircle.Visible = Settings.UseFOV
    if FOVCircle.Visible then
        FOVCircle.Position = Camera.ViewportSize / 2
        FOVCircle.Radius = Settings.FOVRadius
        FOVCircle.Color = Settings.FOVColor
    end

    -- Обновление вида от 3 лица
    updateThirdPersonState()

    -- Поиск цели
    local target = FindSilentTarget()

    -- Auto Fire выстрел
    if Settings.AutoFireEnabled and target then
        Settings.IsFiringState = true
        executeAutoFire()
    else
        Settings.IsFiringState = false
    end

    -- Обновление Anti-Aim (СПИНОЙ ИМЕННО К БЛИЖАЙШЕМУ ВРАГУ)
    updateAntiAimState()

    -- Обновление Чамсов
    updateChams()

    -- Обновление Ночного Режима (Night Bot)
    applyNightState()

    -- ESP обновление игроков
    local camPos = Camera.CFrame.Position

    for _, player in next, Players:GetPlayers() do
        local objects = espInstances[player]
        if player == LocalPlayer then 
            if objects then removeESP(player) end
            continue 
        end

        local char = player.Character
        local humanoid = char and char:FindFirstChildOfClass("Humanoid")
        local rootPart = char and (char:FindFirstChild("HumanoidRootPart") or char:FindFirstChild("UpperTorso") or char.PrimaryPart)
        
        local isDead = char and (char:GetAttribute("Dead") == true or (humanoid and humanoid.Health <= 0))
        
        local allowed = Settings.ESPEnabled and char and rootPart and not isDead
        if allowed and (camPos - rootPart.Position).Magnitude > Settings.ESPMaxDistance then allowed = false end

        if not allowed then
            if objects then
                for _, obj in pairs(objects) do if type(obj) ~= "table" then obj.Visible = false end end
                for _, bone in pairs(objects and objects.Bones or {}) do bone.Visible = false end
            end
            continue
        end

        if not objects then
            createESP(player)
            objects = espInstances[player]
        end

        local playerTeam = get_player_team(player)
        local targetColor = Settings.BoxColor
        if playerTeam and getgenv().ParsaTeamColors[playerTeam] then
            targetColor = getgenv().ParsaTeamColors[playerTeam]
        end

        local rootPos, onScreen = Camera:WorldToViewportPoint(rootPart.Position)
        if onScreen then
            local head = char:FindFirstChild("Head")
            local headPos = head and Camera:WorldToViewportPoint(head.Position) or Camera:WorldToViewportPoint(rootPart.Position + Vector3.new(0, 2, 0))
            local legPos = Camera:WorldToViewportPoint(rootPart.Position - Vector3.new(0, 3, 0))
            
            local height = math.abs(headPos.Y - legPos.Y)
            local width = height / 1.6
            local boxX = rootPos.X - (width / 2)
            local boxY = headPos.Y
            
            objects.Box.Visible = Settings.ESPBoxes
            objects.BoxOutline.Visible = Settings.ESPBoxes
            if Settings.ESPBoxes then
                local size = Vector2.new(width, height)
                local pos = Vector2.new(boxX, boxY)
                objects.Box.Size = size
                objects.Box.Position = pos
                objects.Box.Color = targetColor
                objects.BoxOutline.Size = size
                objects.BoxOutline.Position = pos
            end

            local isR15 = char:FindFirstChild("UpperTorso") ~= nil
            local activeBoneMap = isR15 and BONE_MAP_R15 or BONE_MAP_R6
            
            for i, boneLine in ipairs(objects.Bones) do
                local connection = activeBoneMap[i]
                if Settings.ESPSkeletons and connection then
                    local partA = char:FindFirstChild(connection[1])
                    local partB = char:FindFirstChild(connection[2])
                    if partA and partB then
                        local posA, visA = Camera:WorldToViewportPoint(partA.Position)
                        local posB, visB = Camera:WorldToViewportPoint(partB.Position)
                        if visA and visB then
                            boneLine.From = Vector2.new(posA.X, posA.Y)
                            boneLine.To = Vector2.new(posB.X, posB.Y)
                            boneLine.Color = targetColor
                            boneLine.Visible = true
                        else boneLine.Visible = false end
                    else boneLine.Visible = false end
                else boneLine.Visible = false end
            end

            if Settings.ESPNames then
                objects.Name.Text = player.Name
                objects.Name.Position = Vector2.new(rootPos.X, boxY - 15)
                objects.Name.Visible = true
            else objects.Name.Visible = false end

            if Settings.ESPDistance then
                local distance = math.floor((camPos - rootPart.Position).Magnitude)
                objects.Distance.Text = tostring(distance) .. "m"
                objects.Distance.Position = Vector2.new(rootPos.X, boxY + height + 2)
                objects.Distance.Visible = true
            else objects.Distance.Visible = false end

            if Settings.ESPHealth and humanoid then
                local barPadding, barWidth = 4, 2
                local barX = boxX - barPadding - barWidth
                local healthPercentage = math.clamp(humanoid.Health / humanoid.MaxHealth, 0, 1)
                local healthHeight = height * healthPercentage

                objects.Health.Color = Color3.fromHex("#32CD32"):Lerp(Color3.fromHex("#FF0000"), 1 - healthPercentage)
                objects.HealthOutline.Position = Vector2.new(barX - 1, boxY - 1)
                objects.HealthOutline.Size = Vector2.new(barWidth + 2, height + 2)
                objects.Health.Position = Vector2.new(barX, boxY + (height - healthHeight))
                objects.Health.Size = Vector2.new(barWidth, healthHeight)
                objects.Health.Visible = true
                objects.HealthOutline.Visible = true
            else
                objects.Health.Visible = false
                objects.HealthOutline.Visible = false
            end
        else
            for _, obj in pairs(objects) do if type(obj) ~= "table" then obj.Visible = false end end
            for _, bone in pairs(objects.Bones) do bone.Visible = false end
        end
    end
end)

Players.PlayerRemoving:Connect(removeESP)

-- ==========================================
-- Silent Aim Raycast Хук (Hooking)
-- ==========================================
task.spawn(function()
    if typeof(hookfunction) ~= "function" or typeof(getgc) ~= "function" then
        return
    end

    local bulletClass = nil
    for attempt = 1, 45 do
        for _, obj in next, getgc(true) do
            if type(obj) == "table" and typeof(rawget(obj, "_performRaycast")) == "function" and rawget(obj, "getTrueSpread") ~= nil then
                bulletClass = obj
                break
            end
        end
        if bulletClass then break end
        task.wait(0.5)
    end

    if bulletClass then
        local oldRaycast
        oldRaycast = hookfunction(bulletClass._performRaycast, function(...)
            local returns = table.pack(oldRaycast(...))
            local result = returns[1]
            local SilentTarget = getgenv().ParsaSilentTarget

            if type(result) == "table" then
                pcall(function()
                    local origin = rawget(result, "Origin") or (Workspace.CurrentCamera and Workspace.CurrentCamera.CFrame.Position)
                    local targetPos = nil

                    if Settings.SilentEnabled and SilentTarget and SilentTarget.Parent then
                        targetPos = SilentTarget.Position
                    else
                        local hits = rawget(result, "Hits")
                        if type(hits) == "table" and #hits > 0 and hits[#hits] and hits[#hits].Position then
                            targetPos = hits[#hits].Position
                        elseif rawget(result, "Direction") and origin then
                            targetPos = origin + (result.Direction * (result.Distance or 500))
                        end
                    end

                    if origin and targetPos then
                        spawnBulletTracer(origin, targetPos)
                        spawnHitExplosion(targetPos)
                    end

                    if Settings.SilentEnabled and SilentTarget and SilentTarget.Parent then
                        local hits = rawget(result, "Hits")
                        if type(hits) ~= "table" then return end

                        local aimPos = SilentTarget.Position
                        local lastIndex = nil
                        
                        for index, hit in pairs(hits) do
                            if type(hit) == "table" then
                                if lastIndex == nil or index > lastIndex then lastIndex = index end
                            end
                        end

                        local finalHit = lastIndex and hits[lastIndex]
                        if type(finalHit) ~= "table" then return end

                        finalHit.Instance = SilentTarget
                        finalHit.Position = aimPos
                        if finalHit.Exit ~= nil then finalHit.Exit = false end

                        if typeof(origin) == "Vector3" then
                            local delta = aimPos - origin
                            local length = delta.Magnitude
                            if length > 0.001 then
                                result.Distance = length
                                result.Direction = delta.Unit
                                
                                local unit = delta.Unit
                                for index, hit in pairs(hits) do
                                    if index ~= lastIndex and type(hit) == "table" and typeof(hit.Position) == "Vector3" then
                                        local along = (hit.Position - origin):Dot(unit)
                                        if along > length then
                                            hit.Position = origin + unit * (length * 0.5)
                                        end
                                    end
                                end
                            end
                        end
                    end
                end)
            end
            return table.unpack(returns, 1, returns.n)
        end)
    end
end)
