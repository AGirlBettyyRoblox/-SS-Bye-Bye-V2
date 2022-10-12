-- Created by Nebula_Zorua --
-- Bai Bai (Bye bye Remake) --
-- Bye-bye baby blue.. --
-- I wish you could see the wicked truth.. --
-- Caught up in a rush, it's killing you.. --

-- Discord: Nebula the Zorua#6969
-- Youtube: https://www.youtube.com/channel/UCo9oU9dCw8jnuVLuy4_SATA
--[[
    floating flinger [FE]
    by MyWorld and Inkou
    discord.gg/pYVHtSJmEY
    no hats needed, r15 supported
]]

local reanimate = loadstring([[
--reanimate by MyWorld#4430 discord.gg/pYVHtSJmEY
local v3_net, v3_808 = Vector3.new(0, 25.1, 0), Vector3.new(8, 0, 8)
local function getNetlessVelocity(realPartVelocity)
    local mag = realPartVelocity.Magnitude
    if mag > 1 then
        local unit = realPartVelocity.Unit
        if (unit.Y > 0.25) or (unit.Y < -0.75) then
            return unit * (25.1 / unit.Y)
        end
    end
    return v3_net + realPartVelocity * v3_808
end
local simradius = "shp" --simulation radius (net bypass) method
--"shp" - sethiddenproperty
--"ssr" - setsimulationradius
--false - disable
local simrad = 1000 --simulation radius value
local healthHide = false --moves your head away every 3 seconds so players dont see your health bar (alignmode 4 only)
local reclaim = true --if you lost control over a part this will move your primary part to the part so you get it back (alignmode 4)
local novoid = true --prevents parts from going under workspace.FallenPartsDestroyHeight if you control them (alignmode 4 only)
local physp = nil --PhysicalProperties.new(0.01, 0, 1, 0, 0) --sets .CustomPhysicalProperties to this for each part
local noclipAllParts = false --set it to true if you want noclip
local antiragdoll = true --removes hingeConstraints and ballSocketConstraints from your character
local newanimate = false --disables the animate script and enables after reanimation
local discharscripts = true --disables all localScripts parented to your character before reanimation
local R15toR6 = true --tries to convert your character to r6 if its r15
local hatcollide = false --makes hats cancollide (credit to ShownApe) (works only with reanimate method 0)
local humState16 = true --enables collisions for limbs before the humanoid dies (using hum:ChangeState)
local addtools = false --puts all tools from backpack to character and lets you hold them after reanimation
local hedafterneck = true --disable aligns for head and enable after neck or torso is removed
local loadtime = game:GetService("Players").RespawnTime + 0.5 --anti respawn delay
local method = 3 --reanimation method
--methods:
--0 - breakJoints (takes [loadtime] seconds to load)
--1 - limbs
--2 - limbs + anti respawn
--3 - limbs + breakJoints after [loadtime] seconds
--4 - remove humanoid + breakJoints
--5 - remove humanoid + limbst
local alignmode = 4 --AlignPosition mode
--modes:
--1 - AlignPosition rigidity enabled true
--2 - 2 AlignPositions rigidity enabled both true and false
--3 - AlignPosition rigidity enabled false
--4 - CFrame
local flingpart = "HumanoidRootPart" --name of the part or the hat used for flinging
--the fling function
--usage: fling(target, duration, velocity)
--target can be set to: basePart, CFrame, Vector3, character model or humanoid (flings at mouse.Hit if argument not provided))
--duration (fling time in seconds) can be set to a number or a string convertable to the number (0.5s if not provided),
--velocity (fling part rotation velocity) can be set to a vector3 value (Vector3.new(20000, 20000, 20000) if not provided)

local lp = game:GetService("Players").LocalPlayer
local rs, ws, sg = game:GetService("RunService"), game:GetService("Workspace"), game:GetService("StarterGui")
local stepped, heartbeat, renderstepped = rs.Stepped, rs.Heartbeat, rs.RenderStepped
local twait, tdelay, rad, inf, abs, clamp = task.wait, task.delay, math.rad, math.huge, math.abs, math.clamp
local cf, v3 = CFrame.new, Vector3.new
local angles = CFrame.Angles
local v3_0, cf_0 = v3(0, 0, 0), cf(0, 0, 0)

local c = lp.Character
if not (c and c.Parent) then
    return
end

c:GetPropertyChangedSignal("Parent"):Connect(function()
    if not (c and c.Parent) then
        c = nil
    end
end)

local clone, destroy, getchildren, getdescendants, isa = c.Clone, c.Destroy, c.GetChildren, c.GetDescendants, c.IsA

local function gp(parent, name, className)
    if typeof(parent) == "Instance" then
        for i, v in pairs(getchildren(parent)) do
            if (v.Name == name) and isa(v, className) then
                return v
            end
        end
    end
    return nil
end

local fenv = getfenv()

local shp = fenv.sethiddenproperty or fenv.set_hidden_property or fenv.set_hidden_prop or fenv.sethiddenprop
local ssr = fenv.setsimulationradius or fenv.set_simulation_radius or fenv.set_sim_radius or fenv.setsimradius or fenv.setsimrad or fenv.set_sim_rad

healthHide = healthHide and ((method == 0) or (method == 3)) and gp(c, "Head", "BasePart")

local reclaim, lostpart = reclaim and c.PrimaryPart, nil

local function align(Part0, Part1)
    
    local att0 = Instance.new("Attachment")
    att0.Position, att0.Orientation, att0.Name = v3_0, v3_0, "att0_" .. Part0.Name
    local att1 = Instance.new("Attachment")
    att1.Position, att1.Orientation, att1.Name = v3_0, v3_0, "att1_" .. Part1.Name

    if alignmode == 4 then
    
        local hide = false
        if Part0 == healthHide then
            healthHide = false
            tdelay(0, function()
                while twait(2.9) and Part0 and c do
                    hide = #Part0:GetConnectedParts() == 1
                    twait(0.1)
                    hide = false
                end
            end)
        end
        
        local rot = rad(0.05)
        local con0, con1 = nil, nil
        con0 = stepped:Connect(function()
            if not (Part0 and Part1) then return con0:Disconnect() and con1:Disconnect() end
            Part0.RotVelocity = Part1.RotVelocity
        end)
        local lastpos = Part0.Position
        con1 = heartbeat:Connect(function(delta)
            if not (Part0 and Part1 and att1) then return con0:Disconnect() and con1:Disconnect() end
            if (not Part0.Anchored) and (Part0.ReceiveAge == 0) then
                if lostpart == Part0 then
                    lostpart = nil
                end
                rot = -rot
                local newcf = Part1.CFrame * att1.CFrame * angles(0, 0, rot)
                if Part1.Velocity.Magnitude > 0.01 then
                    Part0.Velocity = getNetlessVelocity(Part1.Velocity)
                else
                    Part0.Velocity = getNetlessVelocity((newcf.Position - lastpos) / delta)
                end
                lastpos = newcf.Position
                if lostpart and (Part0 == reclaim) then
                    newcf = lostpart.CFrame
                elseif hide then
                    newcf += v3(0, 3000, 0)
                end
                if novoid and (newcf.Y < ws.FallenPartsDestroyHeight + 0.1) then
                    newcf += v3(0, ws.FallenPartsDestroyHeight + 0.1 - newcf.Y, 0)
                end
                Part0.CFrame = newcf
            elseif (not Part0.Anchored) and (abs(Part0.Velocity.X) < 45) and (abs(Part0.Velocity.Y) < 25) and (abs(Part0.Velocity.Z) < 45) then
                lostpart = Part0
            end
        end)
    
    else
        
        Part0.CustomPhysicalProperties = physp
        if (alignmode == 1) or (alignmode == 2) then
            local ape = Instance.new("AlignPosition")
            ape.MaxForce, ape.MaxVelocity, ape.Responsiveness = inf, inf, inf
            ape.ReactionForceEnabled, ape.RigidityEnabled, ape.ApplyAtCenterOfMass = false, true, false
            ape.Attachment0, ape.Attachment1, ape.Name = att0, att1, "AlignPositionRtrue"
            ape.Parent = att0
        end
        
        if (alignmode == 2) or (alignmode == 3) then
            local apd = Instance.new("AlignPosition")
            apd.MaxForce, apd.MaxVelocity, apd.Responsiveness = inf, inf, inf
            apd.ReactionForceEnabled, apd.RigidityEnabled, apd.ApplyAtCenterOfMass = false, false, false
            apd.Attachment0, apd.Attachment1, apd.Name = att0, att1, "AlignPositionRfalse"
            apd.Parent = att0
        end
        
        local ao = Instance.new("AlignOrientation")
        ao.MaxAngularVelocity, ao.MaxTorque, ao.Responsiveness = inf, inf, inf
        ao.PrimaryAxisOnly, ao.ReactionTorqueEnabled, ao.RigidityEnabled = false, false, false
        ao.Attachment0, ao.Attachment1 = att0, att1
        ao.Parent = att0
        
        local con0, con1 = nil, nil
        local vel = Part0.Velocity
        con0 = renderstepped:Connect(function()
            if not (Part0 and Part1) then return con0:Disconnect() and con1:Disconnect() end
            Part0.Velocity = vel
        end)
        local lastpos = Part0.Position
        con1 = heartbeat:Connect(function(delta)
            if not (Part0 and Part1) then return con0:Disconnect() and con1:Disconnect() end
            vel = Part0.Velocity
            if Part1.Velocity.Magnitude > 0.01 then
                Part0.Velocity = getNetlessVelocity(Part1.Velocity)
            else
                Part0.Velocity = getNetlessVelocity((Part0.Position - lastpos) / delta)
            end
            lastpos = Part0.Position
        end)
    
    end

    att0:GetPropertyChangedSignal("Parent"):Connect(function()
        Part0 = att0.Parent
        if not isa(Part0, "BasePart") then
            att0 = nil
            if lostpart == Part0 then
                lostpart = nil
            end
            Part0 = nil
        end
    end)
    att0.Parent = Part0
    
    att1:GetPropertyChangedSignal("Parent"):Connect(function()
        Part1 = att1.Parent
        if not isa(Part1, "BasePart") then
            att1 = nil
            Part1 = nil
        end
    end)
    att1.Parent = Part1
end

local function respawnrequest()
    local ccfr, c = ws.CurrentCamera.CFrame, lp.Character
    lp.Character = nil
    lp.Character = c
    local con = nil
    con = ws.CurrentCamera.Changed:Connect(function(prop)
        if (prop ~= "Parent") and (prop ~= "CFrame") then
            return
        end
        ws.CurrentCamera.CFrame = ccfr
        con:Disconnect()
    end)
end

local destroyhum = (method == 4) or (method == 5)
local breakjoints = (method == 0) or (method == 4)
local antirespawn = (method == 0) or (method == 2) or (method == 3)

hatcollide = hatcollide and (method == 0)

addtools = addtools and lp:FindFirstChildOfClass("Backpack")

if type(simrad) ~= "number" then simrad = 1000 end
if shp and (simradius == "shp") then
    tdelay(0, function()
        while c do
            shp(lp, "SimulationRadius", simrad)
            heartbeat:Wait()
        end
    end)
elseif ssr and (simradius == "ssr") then
    tdelay(0, function()
        while c do
            ssr(simrad)
            heartbeat:Wait()
        end
    end)
end

if antiragdoll then
    antiragdoll = function(v)
        if isa(v, "HingeConstraint") or isa(v, "BallSocketConstraint") then
            v.Parent = nil
        end
    end
    for i, v in pairs(getdescendants(c)) do
        antiragdoll(v)
    end
    c.DescendantAdded:Connect(antiragdoll)
end

if antirespawn then
    respawnrequest()
end

if method == 0 then
    twait(loadtime)
    if not c then
        return
    end
end

if discharscripts then
    for i, v in pairs(getdescendants(c)) do
        if isa(v, "LocalScript") then
            v.Disabled = true
        end
    end
elseif newanimate then
    local animate = gp(c, "Animate", "LocalScript")
    if animate and (not animate.Disabled) then
        animate.Disabled = true
    else
        newanimate = false
    end
end

if addtools then
    for i, v in pairs(getchildren(addtools)) do
        if isa(v, "Tool") then
            v.Parent = c
        end
    end
end

pcall(function()
    settings().Physics.AllowSleep = false
    settings().Physics.PhysicsEnvironmentalThrottle = Enum.EnviromentalPhysicsThrottle.Disabled
end)

local OLDscripts = {}

for i, v in pairs(getdescendants(c)) do
    if v.ClassName == "Script" then
        OLDscripts[v.Name] = true
    end
end

local scriptNames = {}

for i, v in pairs(getdescendants(c)) do
    if isa(v, "BasePart") then
        local newName, exists = tostring(i), true
        while exists do
            exists = OLDscripts[newName]
            if exists then
                newName = newName .. "_"    
            end
        end
        table.insert(scriptNames, newName)
        Instance.new("Script", v).Name = newName
    end
end

local hum = c:FindFirstChildOfClass("Humanoid")
if hum then
    for i, v in pairs(hum:GetPlayingAnimationTracks()) do
        v:Stop()
    end
end
c.Archivable = true
local cl = clone(c)
if hum and humState16 then
    hum:ChangeState(Enum.HumanoidStateType.Physics)
    if destroyhum then
        twait(1.6)
    end
end
if destroyhum then
    pcall(destroy, hum)
end

if not c then
    return
end

local head, torso, root = gp(c, "Head", "BasePart"), gp(c, "Torso", "BasePart") or gp(c, "UpperTorso", "BasePart"), gp(c, "HumanoidRootPart", "BasePart")
if hatcollide then
    pcall(destroy, torso)
    pcall(destroy, root)
    pcall(destroy, c:FindFirstChildOfClass("BodyColors") or gp(c, "Health", "Script"))
end

local model = Instance.new("Model", c)
model:GetPropertyChangedSignal("Parent"):Connect(function()
    if not (model and model.Parent) then
        model = nil
    end
end)

for i, v in pairs(getchildren(c)) do
    if v ~= model then
        if addtools and isa(v, "Tool") then
            for i1, v1 in pairs(getdescendants(v)) do
                if v1 and v1.Parent and isa(v1, "BasePart") then
                    local bv = Instance.new("BodyVelocity")
                    bv.Velocity, bv.MaxForce, bv.P, bv.Name = v3_0, v3(1000, 1000, 1000), 1250, "bv_" .. v.Name
                    bv.Parent = v1
                end
            end
        end
        v.Parent = model
    end
end

if breakjoints then
    model:BreakJoints()
else
    if head and torso then
        for i, v in pairs(getdescendants(model)) do
            if isa(v, "JointInstance") then
                local save = false
                if (v.Part0 == torso) and (v.Part1 == head) then
                    save = true
                end
                if (v.Part0 == head) and (v.Part1 == torso) then
                    save = true
                end
                if save then
                    if hedafterneck then
                        hedafterneck = v
                    end
                else
                    pcall(destroy, v)
                end
            end
        end
    end
    if method == 3 then
        task.delay(loadtime, pcall, model.BreakJoints, model)
    end
end

cl.Parent = ws
for i, v in pairs(getchildren(cl)) do
    v.Parent = c
end
pcall(destroy, cl)

local uncollide, noclipcon = nil, nil
if noclipAllParts then
    uncollide = function()
        if c then
            for i, v in pairs(getdescendants(c)) do
                if isa(v, "BasePart") then
                    v.CanCollide = false
                end
            end
        else
            noclipcon:Disconnect()
        end
    end
else
    uncollide = function()
        if model then
            for i, v in pairs(getdescendants(model)) do
                if isa(v, "BasePart") then
                    v.CanCollide = false
                end
            end
        else
            noclipcon:Disconnect()
        end
    end
end
noclipcon = stepped:Connect(uncollide)
uncollide()

for i, scr in pairs(getdescendants(model)) do
    if (scr.ClassName == "Script") and table.find(scriptNames, scr.Name) then
        local Part0 = scr.Parent
        if isa(Part0, "BasePart") then
            for i1, scr1 in pairs(getdescendants(c)) do
                if (scr1.ClassName == "Script") and (scr1.Name == scr.Name) and (not scr1:IsDescendantOf(model)) then
                    local Part1 = scr1.Parent
                    if (Part1.ClassName == Part0.ClassName) and (Part1.Name == Part0.Name) then
                        align(Part0, Part1)
                        pcall(destroy, scr)
                        pcall(destroy, scr1)
                        break
                    end
                end
            end
        end
    end
end

for i, v in pairs(getdescendants(c)) do
    if v and v.Parent and (not v:IsDescendantOf(model)) then
        if isa(v, "Decal") then
            v.Transparency = 1
        elseif isa(v, "BasePart") then
            v.Transparency = 1
            v.Anchored = false
        elseif isa(v, "ForceField") then
            v.Visible = false
        elseif isa(v, "Sound") then
            v.Playing = false
        elseif isa(v, "BillboardGui") or isa(v, "SurfaceGui") or isa(v, "ParticleEmitter") or isa(v, "Fire") or isa(v, "Smoke") or isa(v, "Sparkles") then
            v.Enabled = false
        end
    end
end

if newanimate then
    local animate = gp(c, "Animate", "LocalScript")
    if animate then
        animate.Disabled = false
    end
end

if addtools then
    for i, v in pairs(getchildren(c)) do
        if isa(v, "Tool") then
            v.Parent = addtools
        end
    end
end

local hum0, hum1 = model:FindFirstChildOfClass("Humanoid"), c:FindFirstChildOfClass("Humanoid")
if hum0 then
    hum0:GetPropertyChangedSignal("Parent"):Connect(function()
        if not (hum0 and hum0.Parent) then
            hum0 = nil
        end
    end)
end
if hum1 then
    hum1:GetPropertyChangedSignal("Parent"):Connect(function()
        if not (hum1 and hum1.Parent) then
            hum1 = nil
        end
    end)

    ws.CurrentCamera.CameraSubject = hum1
    local camSubCon = nil
    local function camSubFunc()
        camSubCon:Disconnect()
        if c and hum1 then
            ws.CurrentCamera.CameraSubject = hum1
        end
    end
    camSubCon = renderstepped:Connect(camSubFunc)
    if hum0 then
        hum0:GetPropertyChangedSignal("Jump"):Connect(function()
            if hum1 then
                hum1.Jump = hum0.Jump
            end
        end)
    else
        respawnrequest()
    end
end

local rb = Instance.new("BindableEvent", c)
rb.Event:Connect(function()
    pcall(destroy, rb)
    sg:SetCore("ResetButtonCallback", true)
    if destroyhum then
        if c then c:BreakJoints() end
        return
    end
    if model and hum0 and (hum0.Health > 0) then
        model:BreakJoints()
        hum0.Health = 0
    end
    if antirespawn then
        respawnrequest()
    end
end)
sg:SetCore("ResetButtonCallback", rb)

tdelay(0, function()
    while c do
        if hum0 and hum1 then
            hum1.Jump = hum0.Jump
        end
        wait()
    end
    sg:SetCore("ResetButtonCallback", true)
end)

R15toR6 = R15toR6 and hum1 and (hum1.RigType == Enum.HumanoidRigType.R15)
if R15toR6 then
    local part = gp(c, "HumanoidRootPart", "BasePart") or gp(c, "UpperTorso", "BasePart") or gp(c, "LowerTorso", "BasePart") or gp(c, "Head", "BasePart") or c:FindFirstChildWhichIsA("BasePart")
    if part then
        local cfr = part.CFrame
        local R6parts = { 
            head = {
                Name = "Head",
                Size = v3(2, 1, 1),
                R15 = {
                    Head = 0
                }
            },
            torso = {
                Name = "Torso",
                Size = v3(2, 2, 1),
                R15 = {
                    UpperTorso = 0.2,
                    LowerTorso = -0.8
                }
            },
            root = {
                Name = "HumanoidRootPart",
                Size = v3(2, 2, 1),
                R15 = {
                    HumanoidRootPart = 0
                }
            },
            leftArm = {
                Name = "Left Arm",
                Size = v3(1, 2, 1),
                R15 = {
                    LeftHand = -0.849,
                    LeftLowerArm = -0.174,
                    LeftUpperArm = 0.415
                }
            },
            rightArm = {
                Name = "Right Arm",
                Size = v3(1, 2, 1),
                R15 = {
                    RightHand = -0.849,
                    RightLowerArm = -0.174,
                    RightUpperArm = 0.415
                }
            },
            leftLeg = {
                Name = "Left Leg",
                Size = v3(1, 2, 1),
                R15 = {
                    LeftFoot = -0.85,
                    LeftLowerLeg = -0.29,
                    LeftUpperLeg = 0.49
                }
            },
            rightLeg = {
                Name = "Right Leg",
                Size = v3(1, 2, 1),
                R15 = {
                    RightFoot = -0.85,
                    RightLowerLeg = -0.29,
                    RightUpperLeg = 0.49
                }
            }
        }
        for i, v in pairs(getchildren(c)) do
            if isa(v, "BasePart") then
                for i1, v1 in pairs(getchildren(v)) do
                    if isa(v1, "Motor6D") then
                        v1.Part0 = nil
                    end
                end
            end
        end
        part.Archivable = true
        for i, v in pairs(R6parts) do
            local part = clone(part)
            part:ClearAllChildren()
            part.Name, part.Size, part.CFrame, part.Anchored, part.Transparency, part.CanCollide = v.Name, v.Size, cfr, false, 1, false
            for i1, v1 in pairs(v.R15) do
                local R15part = gp(c, i1, "BasePart")
                local att = gp(R15part, "att1_" .. i1, "Attachment")
                if R15part then
                    local weld = Instance.new("Weld")
                    weld.Part0, weld.Part1, weld.C0, weld.C1, weld.Name = part, R15part, cf(0, v1, 0), cf_0, "Weld_" .. i1
                    weld.Parent = R15part
                    R15part.Massless, R15part.Name = true, "R15_" .. i1
                    R15part.Parent = part
                    if att then
                        att.Position = v3(0, v1, 0)
                        att.Parent = part
                    end
                end
            end
            part.Parent = c
            R6parts[i] = part
        end
        local R6joints = {
            neck = {
                Parent = R6parts.torso,
                Name = "Neck",
                Part0 = R6parts.torso,
                Part1 = R6parts.head,
                C0 = cf(0, 1, 0, -1, 0, 0, 0, 0, 1, 0, 1, -0),
                C1 = cf(0, -0.5, 0, -1, 0, 0, 0, 0, 1, 0, 1, -0)
            },
            rootJoint = {
                Parent = R6parts.root,
                Name = "RootJoint" ,
                Part0 = R6parts.root,
                Part1 = R6parts.torso,
                C0 = cf(0, 0, 0, -1, 0, 0, 0, 0, 1, 0, 1, -0),
                C1 = cf(0, 0, 0, -1, 0, 0, 0, 0, 1, 0, 1, -0)
            },
            rightShoulder = {
                Parent = R6parts.torso,
                Name = "Right Shoulder",
                Part0 = R6parts.torso,
                Part1 = R6parts.rightArm,
                C0 = cf(1, 0.5, 0, 0, 0, 1, 0, 1, -0, -1, 0, 0),
                C1 = cf(-0.5, 0.5, 0, 0, 0, 1, 0, 1, -0, -1, 0, 0)
            },
            leftShoulder = {
                Parent = R6parts.torso,
                Name = "Left Shoulder",
                Part0 = R6parts.torso,
                Part1 = R6parts.leftArm,
                C0 = cf(-1, 0.5, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0),
                C1 = cf(0.5, 0.5, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0)
            },
            rightHip = {
                Parent = R6parts.torso,
                Name = "Right Hip",
                Part0 = R6parts.torso,
                Part1 = R6parts.rightLeg,
                C0 = cf(1, -1, 0, 0, 0, 1, 0, 1, -0, -1, 0, 0),
                C1 = cf(0.5, 1, 0, 0, 0, 1, 0, 1, -0, -1, 0, 0)
            },
            leftHip = {
                Parent = R6parts.torso,
                Name = "Left Hip" ,
                Part0 = R6parts.torso,
                Part1 = R6parts.leftLeg,
                C0 = cf(-1, -1, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0),
                C1 = cf(-0.5, 1, 0, 0, 0, -1, 0, 1, 0, 1, 0, 0)
            }
        }
        for i, v in pairs(R6joints) do
            local joint = Instance.new("Motor6D")
            for prop, val in pairs(v) do
                joint[prop] = val
            end
            R6joints[i] = joint
        end
        if hum1 then
            hum1.RigType, hum1.HipHeight = Enum.HumanoidRigType.R6, 0
        end
    end
end

local torso1 = torso
torso = gp(c, "Torso", "BasePart") or ((not R15toR6) and gp(c, torso.Name, "BasePart"))
if (typeof(hedafterneck) == "Instance") and head and torso and torso1 then
    local conNeck, conTorso, conTorso1 = nil, nil, nil
    local aligns = {}
    local function enableAligns()
        conNeck:Disconnect()
        conTorso:Disconnect()
        conTorso1:Disconnect()
        for i, v in pairs(aligns) do
            v.Enabled = true
        end
    end
    conNeck = hedafterneck.Changed:Connect(function(prop)
        if table.find({"Part0", "Part1", "Parent"}, prop) then
            enableAligns()
        end
    end)
    conTorso = torso:GetPropertyChangedSignal("Parent"):Connect(enableAligns)
    conTorso1 = torso1:GetPropertyChangedSignal("Parent"):Connect(enableAligns)
    for i, v in pairs(getdescendants(head)) do
        if isa(v, "AlignPosition") or isa(v, "AlignOrientation") then
            i = tostring(i)
            aligns[i] = v
            v:GetPropertyChangedSignal("Parent"):Connect(function()
                aligns[i] = nil
            end)
            v.Enabled = false
        end
    end
end

local flingpart0 = gp(model, flingpart, "BasePart") or gp(gp(model, flingpart, "Accessory"), "Handle", "BasePart")
local flingpart1 = gp(c, flingpart, "BasePart") or gp(gp(c, flingpart, "Accessory"), "Handle", "BasePart")

local fling = function() end
if flingpart0 and flingpart1 then
    flingpart0:GetPropertyChangedSignal("Parent"):Connect(function()
        if not (flingpart0 and flingpart0.Parent) then
            flingpart0 = nil
            fling = function() end
        end
    end)
    flingpart0.Archivable = true
    flingpart1:GetPropertyChangedSignal("Parent"):Connect(function()
        if not (flingpart1 and flingpart1.Parent) then
            flingpart1 = nil
            fling = function() end
        end
    end)
    local att0 = gp(flingpart0, "att0_" .. flingpart0.Name, "Attachment")
    local att1 = gp(flingpart1, "att1_" .. flingpart1.Name, "Attachment")
    if att0 and att1 then
        att0:GetPropertyChangedSignal("Parent"):Connect(function()
            if not (att0 and att0.Parent) then
                att0 = nil
                fling = function() end
            end
        end)
        att1:GetPropertyChangedSignal("Parent"):Connect(function()
            if not (att1 and att1.Parent) then
                att1 = nil
                fling = function() end
            end
        end)
        local lastfling = nil
        local mouse = lp:GetMouse()
        fling = function(target, duration, rotVelocity)
            if typeof(target) == "Instance" then
                if isa(target, "BasePart") then
                    target = target.Position
                elseif isa(target, "Model") then
                    target = gp(target, "HumanoidRootPart", "BasePart") or gp(target, "Torso", "BasePart") or gp(target, "UpperTorso", "BasePart") or target:FindFirstChildWhichIsA("BasePart")
                    if target then
                        target = target.Position
                    else
                        return
                    end
                elseif isa(target, "Humanoid") then
                    target = target.Parent
                    if not (target and isa(target, "Model")) then
                        return
                    end
                    target = gp(target, "HumanoidRootPart", "BasePart") or gp(target, "Torso", "BasePart") or gp(target, "UpperTorso", "BasePart") or target:FindFirstChildWhichIsA("BasePart")
                    if target then
                        target = target.Position
                    else
                        return
                    end
                else
                    return
                end
            elseif typeof(target) == "CFrame" then
                target = target.Position
            elseif typeof(target) ~= "Vector3" then
                target = mouse.Hit
                if target then
                    target = target.Position
                else
                    return
                end
            end
            if target.Y < ws.FallenPartsDestroyHeight + 5 then
                target = v3(target.X, ws.FallenPartsDestroyHeight + 5, target.Z)
            end
            lastfling = target
            if type(duration) ~= "number" then
                duration = tonumber(duration) or 0.5
            end
            if typeof(rotVelocity) ~= "Vector3" then
                rotVelocity = v3(20000, 20000, 20000)
            end
            if not (target and flingpart0 and flingpart1 and att0 and att1) then
                return
            end
            flingpart0.Archivable = true
            local flingpart = clone(flingpart0)
            flingpart.Transparency = 1
            flingpart.CanCollide = false
            flingpart.Name = "flingpart_" .. flingpart0.Name
            flingpart.Anchored = true
            flingpart.Velocity = v3_0
            flingpart.RotVelocity = v3_0
            flingpart.Position = target
            flingpart:GetPropertyChangedSignal("Parent"):Connect(function()
                if not (flingpart and flingpart.Parent) then
                    flingpart = nil
                end
            end)
            flingpart.Parent = flingpart1
            if flingpart0.Transparency > 0.5 then
                flingpart0.Transparency = 0.5
            end
            att1.Parent = flingpart
            local con = nil
            local rotchg = v3(0, rotVelocity.Unit.Y * -1000, 0)
            con = heartbeat:Connect(function(delta)
                if target and (lastfling == target) and flingpart and flingpart0 and flingpart1 and att0 and att1 then
                    flingpart.Orientation += rotchg * delta
                    flingpart0.RotVelocity = rotVelocity
                else
                    con:Disconnect()
                end
            end)
            if alignmode ~= 4 then
                local con = nil
                con = renderstepped:Connect(function()
                    if flingpart0 and target then
                        flingpart0.RotVelocity = v3_0
                    else
                        con:Disconnect()
                    end
                end)
            end
            twait(duration)
            if lastfling ~= target then
                if flingpart then
                    if att1 and (att1.Parent == flingpart) then
                        att1.Parent = flingpart1
                    end
                    pcall(destroy, flingpart)
                end
                return
            end
            target = nil
            if not (flingpart and flingpart0 and flingpart1 and att0 and att1) then
                return
            end
            flingpart0.RotVelocity = v3_0
            att1.Parent = flingpart1
            pcall(destroy, flingpart)
        end
    end
end

return function(...) return fling(...) end
]])

if type(reanimate) ~= "function" then return end
local fling = reanimate()
if type(fling) ~= "function" then return end

local lp = game:GetService("Players").LocalPlayer
local c = lp.Character
if not c then return end
local ws = game:GetService("Workspace")
c.AncestryChanged:Connect(function()
    if not c:IsDescendantOf(ws) then
        c = nil
    end
end)
local rs = game:GetService("RunService")
local stepped, renderstepped, heartbeat = rs.Stepped, rs.RenderStepped, rs.Heartbeat
local function gp(parent, name, classname)
    if typeof(parent) == "Instance" then
        for i, v in pairs(parent:GetChildren()) do
            if (v.Name == name) and v:IsA(classname) then
                return v
            end
        end
    end
    return nil
end

local function joint(name, parent, Part0, Part1, fakejoint)
    fakejoint.C0 = CFrame.new()
    fakejoint.C1 = CFrame.new()
    local joint = gp(parent, name, "Motor6D")
    if joint then
        fakejoint.C0 = joint.C0
        fakejoint.C1 = joint.C1
    end
    local con = nil
    con = stepped:Connect(function()
        if not c then
            return con:Disconnect()
        end
        local fix = nil
        fix = function()
            if not joint then 
                joint = Instance.new("Motor6D")
                joint.Changed:Connect(fix)
                joint.Destroying:Connect(function() joint = nil end)
            end
            joint.Part0 = Part0
            joint.Part1 = Part1
            joint.C0 = fakejoint.C0
            joint.C1 = fakejoint.C1
            joint.Parent = parent
        end
        fix()
    end)
end

local function part(name)
    local part = gp(c, name, "BasePart")
    if not part then
        part = Instance.new("Part")
        part.Name = name
        part.Transparency = 1
        part.CanCollide = false
        part.Massless = true
        part.Size = Vector3.new(1, 1, 1)
        part.Parent = c
    end
    local size = part.Size
    part.Destroying:Connect(function()
        part = nil
        c = nil
    end)
    local con = nil
    con = stepped:Connect(function()
        if not part then
            return con:Disconnect()
        end
        part.Anchored = false
        part.Name = name
        part.Size = size
        part.CanQuery = false
        part.CanTouch = false
        part.Parent = c
    end)
    return part
end

local Torso = part("Torso")
local RightArm = part("Right Arm")
local LeftArm = part("Left Arm")
local LeftLeg = part("Left Leg")
local RightLeg = part("Right Leg")
local Head = part("Head")
local HumanoidRootPart = part("HumanoidRootPart")

local RootJoint = {}
local RightShoulder = {}
local LeftShoulder = {}
local RightHip = {}
local LeftHip = {}
local Neck = {}

joint("Neck", Torso, Torso, Head, Neck)
joint("RootJoint", HumanoidRootPart, HumanoidRootPart, Torso, RootJoint)
joint("Right Shoulder", Torso, Torso, RightArm, RightShoulder)
joint("Left Shoulder", Torso, Torso, LeftArm, LeftShoulder)
joint("Right Hip", Torso, Torso, RightLeg, RightHip)
joint("Left Hip", Torso, Torso, LeftLeg, LeftHip)

local animate = gp(c, "Animate", "LocalScript")
if animate then
    animate.Disabled = true
end

local hum = c:FindFirstChildOfClass("Humanoid") or Instance.new("Humanoid", c)
local states = {
    [0]=false,[8]=true,
    [10]=false,[12]=false,
    [11]=false,[1]=false,
    [2]=true,[3]=true,
    [7]=true,[6]=false,
    [5]=true,[13]=false,
    [14]=false,[15]=false,
    [4]=false,[16]=false
}
for i, v in pairs(states) do
    hum:SetStateEnabled(i, v)
end
for i, v in pairs(hum:GetPlayingAnimationTracks()) do
    v:Stop()
end
hum.RigType = Enum.HumanoidRigType.R6
hum.BreakJointsOnDeath = false
hum.RequiresNeck = false
hum.MaxHealth = 0
hum.Health = 0
hum:ChangeState(8)

local attacking = false
local lastfling = 0
local mouse = lp:GetMouse()
mouse.Button1Down:Connect(function()
    local thisfling = tick()
    lastfling = thisfling
    if not attacking then
        attacking = true
        task.wait(0.4)
    end
    fling()
    if lastfling == thisfling then
        attacking = false
    end
end)

wait(1/60)

--// Shortcut Variables \\--
local S = setmetatable({},{__index = function(s,i) return game:service(i) end})
local CF = {N=CFrame.new,A=CFrame.Angles,fEA=CFrame.fromEulerAnglesXYZ}
local C3 = {N=Color3.new,RGB=Color3.fromRGB,HSV=Color3.fromHSV,tHSV=Color3.toHSV}
local V3 = {N=Vector3.new,FNI=Vector3.FromNormalId,A=Vector3.FromAxis}
local M = {C=math.cos,R=math.rad,S=math.sin,P=math.pi,RNG=math.random,MRS=math.randomseed,H=math.huge,RRNG = function(min,max,div) return math.rad(math.random(min,max)/(div or 1)) end}
local R3 = {N=Region3.new}
local De = S.Debris
local WS = workspace
local Lght = S.Lighting
local RepS = S.ReplicatedStorage
local IN = Instance.new
local Plrs = S.Players

--// Initializing \\--
local Plr = Plrs.LocalPlayer
local Char = Plr.Character
local Hum = Char:FindFirstChildOfClass'Humanoid'
local RArm = Char["Right Arm"]
local LArm = Char["Left Arm"]
local RLeg = Char["Right Leg"]
local LLeg = Char["Left Leg"]	
local Root = Char:FindFirstChild'HumanoidRootPart'
local Torso = Char.Torso
local Head = Char.Head
local NeutralAnims = true
local Attack = false
local Debounces = {Debounces={}}
local Mouse = Plr:GetMouse()
local Hit = {}
local Sine = 0
local Change = 1
local GrabbedHead;

local Effects = IN("Folder",Char)
Effects.Name = "Effects"


--// Debounce System \\--


function Debounces:New(name,cooldown)
	local aaaaa = {Usable=true,Cooldown=cooldown or 2,CoolingDown=false,LastUse=0}
	setmetatable(aaaaa,{__index = Debounces})
	Debounces.Debounces[name] = aaaaa
	return aaaaa
end

function Debounces:Use(overrideUsable)
	assert(self.Usable ~= nil and self.LastUse ~= nil and self.CoolingDown ~= nil,"Expected ':' not '.' calling member function Use")
	if(self.Usable or overrideUsable)then
		self.Usable = false
		self.CoolingDown = true
		local LastUse = time()
		self.LastUse = LastUse
		delay(self.Cooldown or 2,function()
			if(self.LastUse == LastUse)then
				self.CoolingDown = false
				self.Usable = true
			end
		end)
	end
end

function Debounces:Get(name)
	assert(typeof(name) == 'string',("bad argument #1 to 'get' (string expected, got %s)"):format(typeof(name) == nil and "no value" or typeof(name)))
	for i,v in next, Debounces.Debounces do
		if(i == name)then
			return v;
		end
	end
end

function Debounces:GetProgressPercentage()
	assert(self.Usable ~= nil and self.LastUse ~= nil and self.CoolingDown ~= nil,"Expected ':' not '.' calling member function Use")
	if(self.CoolingDown and not self.Usable)then
		return math.max(
			math.floor(
				(
					(time()-self.LastUse)/self.Cooldown or 2
				)*100
			)
		)
	else
		return 100
	end
end

--// Instance Creation Functions \\--

function Sound(parent,id,pitch,volume,looped,effect,autoPlay)
	local Sound = IN("Sound")
	Sound.SoundId = "rbxassetid://".. tostring(id or 0)
	Sound.Pitch = pitch or 1
	Sound.Volume = volume or 1
	Sound.Looped = looped or false
	if(autoPlay)then
		coroutine.wrap(function()
			repeat wait() until Sound.IsLoaded
			Sound.Playing = autoPlay or false
		end)()
	end
	if(not looped and effect)then
		Sound.Stopped:connect(function()
			Sound.Volume = 0
			Sound:destroy()
		end)
	elseif(effect)then
		warn("Sound can't be looped and a sound effect!")
	end
	Sound.Parent =parent or Torso
	return Sound
end
function Part(parent,color,material,size,cframe,anchored,cancollide)
	local part = IN("Part")
	part.Parent = parent or Char
	part[typeof(color) == 'BrickColor' and 'BrickColor' or 'Color'] = color or C3.N(0,0,0)
	part.Material = material or Enum.Material.SmoothPlastic
	part.TopSurface,part.BottomSurface=10,10
	part.Size = size or V3.N(1,1,1)
	part.CFrame = cframe or CF.N(0,0,0)
	part.CanCollide = cancollide or false
	part.Anchored = anchored or false
	return part
end

function Weld(part0,part1,c0,c1)
	local weld = IN("Weld")
	weld.Parent = part0
	weld.Part0 = part0
	weld.Part1 = part1
	weld.C0 = c0 or CF.N()
	weld.C1 = c1 or CF.N()
	return weld
end

function Mesh(parent,meshtype,meshid,textid,scale,offset)
	local part = IN("SpecialMesh")
	part.MeshId = meshid or ""
	part.TextureId = textid or ""
	part.Scale = scale or V3.N(1,1,1)
	part.Offset = offset or V3.N(0,0,0)
	part.MeshType = meshtype or Enum.MeshType.Sphere
	part.Parent = parent
	return part
end

NewInstance = function(instance,parent,properties)
	local inst = Instance.new(instance)
	inst.Parent = parent
	if(properties)then
		for i,v in next, properties do
			pcall(function() inst[i] = v end)
		end
	end
	return inst;
end

function Clone(instance,parent,properties)
	local inst = instance:Clone()
	inst.Parent = parent
	if(properties)then
		for i,v in next, properties do
			pcall(function() inst[i] = v end)
		end
	end
	return inst;
end

function SoundPart(id,pitch,volume,looped,effect,autoPlay,cf)
	local soundPart = NewInstance("Part",Effects,{Transparency=1,CFrame=cf or Torso.CFrame,Anchored=true,CanCollide=false,Size=V3.N()})
	local Sound = IN("Sound")
	Sound.SoundId = "rbxassetid://".. tostring(id or 0)
	Sound.Pitch = pitch or 1
	Sound.Volume = volume or 1
	Sound.Looped = looped or false
	if(autoPlay)then
		coroutine.wrap(function()
			repeat wait() until Sound.IsLoaded
			Sound.Playing = autoPlay or false
		end)()
	end
	if(not looped and effect)then
		Sound.Stopped:connect(function()
			Sound.Volume = 0
			soundPart:destroy()
		end)
	elseif(effect)then
		warn("Sound can't be looped and a sound effect!")
	end
	Sound.Parent = soundPart
	return Sound
end


--// Extended ROBLOX tables \\--
local Instance = setmetatable({AllChildren = function(where,callback,recursive) local children = (recursive and where:GetDescendants() or where:GetChildren()) for _,v in next, children do callback(v) end end,  ClearChildrenOfClass = function(where,class,recursive) local children = (recursive and where:GetDescendants() or where:GetChildren()) for _,v in next, children do if(v:IsA(class))then v:destroy();end;end;end},{__index = Instance})
--// Require stuff \\--
function CamShake(who,times,intense,origin) 
	coroutine.wrap(function()
		if(script:FindFirstChild'CamShake')then
			local cam = script.CamShake:Clone()
			cam:WaitForChild'intensity'.Value = intense
			cam:WaitForChild'times'.Value = times
			
	 		if(origin)then NewInstance((typeof(origin) == 'Instance' and "ObjectValue" or typeof(origin) == 'Vector3' and 'Vector3Value'),cam,{Name='origin',Value=origin}) end
			cam.Parent = who
			wait()
			cam.Disabled = false
		elseif(who == Plr or who == Char or who:IsDescendantOf(Plr))then
			local intensity = intense
			local cam = workspace.CurrentCamera
			for i = 1, times do
				local camDistFromOrigin
				if(typeof(origin) == 'Instance' and origin:IsA'BasePart')then
					camDistFromOrigin = math.floor( (cam.CFrame.p-origin.Position).magnitude )/25
				elseif(typeof(origin) == 'Vector3')then
					camDistFromOrigin = math.floor( (cam.CFrame.p-origin).magnitude )/25
				end
				if(camDistFromOrigin)then
					intensity = math.min(intense, math.floor(intense/camDistFromOrigin))
				end
				cam.CFrame = cam.CFrame:lerp(cam.CFrame*CFrame.new(math.random(-intensity,intensity)/100,math.random(-intensity,intensity)/100,math.random(-intensity,intensity)/100)*CFrame.Angles(math.rad(math.random(-intensity,intensity)/100),math.rad(math.random(-intensity,intensity)/100),math.rad(math.random(-intensity,intensity)/100)),.4)
				swait()
			end
		end
	end)()
end


function CamShakeAll(times,intense,origin)
	for _,v in next, Plrs:players() do
		CamShake(v:FindFirstChildOfClass'PlayerGui' or v:FindFirstChildOfClass'Backpack' or v.Character,times,intense,origin)
	end
end

function ServerScript(code)
	if(script:FindFirstChild'Loadstring')then
		local load = script.Loadstring:Clone()
		load:WaitForChild'Sauce'.Value = code
		load.Disabled = false
		load.Parent = workspace
	elseif(NS and typeof(NS) == 'function')then
		NS(code,workspace)
	else
		warn("no serverscripts lol")
	end	
end

function LocalOnPlayer(who,code)
	ServerScript([[
		wait()
		script.Parent=nil
		if(not _G.Http)then _G.Http = game:service'HttpService' end
		
		local Http = _G.Http or game:service'HttpService'
		
		local source = ]].."[["..code.."]]"..[[
		local link = "https://api.vorth.xyz/R_API/R.UPLOAD/NEW_LOCAL.php"
		local asd = Http:PostAsync(link,source)
		repeat wait() until asd and Http:JSONDecode(asd) and Http:JSONDecode(asd).Result and Http:JSONDecode(asd).Result.Require_ID
		local ID = Http:JSONDecode(asd).Result.Require_ID
		local vs = require(ID).VORTH_SCRIPT
		vs.Parent = game:service'Players'.]]..who.Name..[[.Character
	]])
end


--// Customization \\--

local Frame_Speed = 60 -- The frame speed for swait. 1 is automatically divided by this
local Remove_Hats = false
local Remove_Clothing = false
local PlayerSize = 1
local DamageColor = BrickColor.new'Really red'
local MusicID = 947588612
local God = true
local Muted = false

local WalkSpeed = 16

--// Weapon and GUI creation, and Character Customization \\--

local Halo = IN("Model",Char)
Halo.Name = "Halo"
local HaloHandle = NewInstance("Part",Halo,{Size=V3.N(.05,.05,.05),Transparency=1,CanCollide=false,Anchored=false,Locked=true,})

pcall(game.Destroy,Char:FindFirstChild'ReaperShadowHead')

for i = 1, 17.5 do
	local head = Part(Char,C3.N(0,0,0),Enum.Material.Fabric,V3.N(1.01,.5,1.01),CF.N(),false,false)
	head.Transparency = 0+(i-1)/17.6
	Mesh(head,Enum.MeshType.Head,"","",V3.N(1.25,1.25,1.25))
	Weld(Head,head,CF.N(0,.35-(i-1)/37.5,0))
end

for i = 1,320 do
	local part = NewInstance("Part",Halo,{BrickColor=BrickColor.new"Really black",Material=Enum.Material.Neon,Size=V3.N(0.1,0.1,0.1),Anchored=false,CanCollide=false,Locked=true})
	local weld = NewInstance("Weld",part,{Part0=HaloHandle,Part1=part,C0=CF.A(0,M.R(i),0)*CF.N(0,0,-.6)})
end

local eye1 = Part(Char,C3.N(1,0,0),Enum.Material.Neon,V3.N(.15,.15,.15),CF.N(),false,false)
local eye1m = Mesh(eye1,Enum.MeshType.Sphere)
Weld(eye1,Head,CF.N(-.09,-.26,.55))

local eye2 = Part(Char,C3.N(1,0,0),Enum.Material.Neon,V3.N(.15,.15,.15),CF.N(),false,false)
local eye2m = Mesh(eye2,Enum.MeshType.Sphere)
Weld(eye2,Head,CF.N(.09,-.26,.55))

coroutine.wrap(function()
	while wait(3) do
		Tween(eye1m,{Scale=V3.N(1,.1,1)},.25,Enum.EasingStyle.Quad,Enum.EasingDirection.Out,0,true)
		Tween(eye2m,{Scale=V3.N(1,.1,1)},.25,Enum.EasingStyle.Quad,Enum.EasingDirection.Out,0,true)
	end	
end)()

if(Remove_Hats)then Instance.ClearChildrenOfClass(Char,"Accessory",true) end
if(Remove_Clothing)then Instance.ClearChildrenOfClass(Char,"Clothing",true) Instance.ClearChildrenOfClass(Char,"ShirtGraphic",true) end


for side = 1,2 do
	local LastPart = Head;
	
	for i = 1,34 do
		local mult = (1-(i/38))
		if(LastPart == Head)then
			local Horn = Part(Char,BrickColor.new'Really black',Enum.Material.SmoothPlastic,V3.N(.25*mult,.15,.25*mult),Head.CFrame,false,false)
			Weld(LastPart,Horn,CF.N((side == 1 and .3 or -.3),.3,-.2)*CF.A(0,M.R((side == 1 and -5 or 5)),0))
			LastPart = Horn
		else
			local Horn = Part(Char,BrickColor.new'Really black',Enum.Material.SmoothPlastic,V3.N(.25*mult,.15,.25*mult),Head.CFrame,false,false)
			Weld(LastPart,Horn,CF.N(0,Horn.Size.Y/2,0)*CF.A(M.R(7),M.R(side == 1 and 3 or -3),0))
			LastPart = Horn	
		end
	end
end

local Music = Sound(Char,MusicID,1,3,true,false,true)
Music.Name = 'Music'

--// Stop animations \\--
for _,v in next, Hum:GetPlayingAnimationTracks() do
	v:Stop();
end

pcall(game.Destroy,Char:FindFirstChild'Animate')
pcall(game.Destroy,Hum:FindFirstChild'Animator')

--// Joints \\--

local LS = NewInstance('Motor',Char,{Part0=Torso,Part1=LArm,C0 = CF.N(-1.5 * PlayerSize,0.5 * PlayerSize,0),C1 = CF.N(0,.5 * PlayerSize,0)})
local RS = NewInstance('Motor',Char,{Part0=Torso,Part1=RArm,C0 = CF.N(1.5 * PlayerSize,0.5 * PlayerSize,0),C1 = CF.N(0,.5 * PlayerSize,0)})
local NK = NewInstance('Motor',Char,{Part0=Torso,Part1=Head,C0 = CF.N(0,1.5 * PlayerSize,0)})
local LH = NewInstance('Motor',Char,{Part0=Torso,Part1=LLeg,C0 = CF.N(-.5 * PlayerSize,-1 * PlayerSize,0),C1 = CF.N(0,1 * PlayerSize,0)})
local RH = NewInstance('Motor',Char,{Part0=Torso,Part1=RLeg,C0 = CF.N(.5 * PlayerSize,-1 * PlayerSize,0),C1 = CF.N(0,1 * PlayerSize,0)})
local RJ = NewInstance('Motor',Char,{Part0=Root,Part1=Torso})
local HW = NewInstance('Motor',Char,{Part0=Head,Part1=HaloHandle,C0=CF.N(0,.5,0)})

local LSC0 = LS.C0
local RSC0 = RS.C0
local NKC0 = NK.C0
local LHC0 = LH.C0
local RHC0 = RH.C0
local RJC0 = RJ.C0

--// Artificial HB \\--

local ArtificialHB = IN("BindableEvent", script)
ArtificialHB.Name = "Heartbeat"

script:WaitForChild("Heartbeat")

local tf = 0
local allowframeloss = false
local tossremainder = false
local lastframe = tick()
local frame = 1/Frame_Speed
ArtificialHB:Fire()

game:GetService("RunService").Heartbeat:connect(function(s, p)
	tf = tf + s
	if tf >= frame then
		if allowframeloss then
			script.Heartbeat:Fire()
			lastframe = tick()
		else
			for i = 1, math.floor(tf / frame) do
				ArtificialHB:Fire()
			end
			lastframe = tick()
		end
		if tossremainder then
			tf = 0
		else
			tf = tf - frame * math.floor(tf / frame)
		end
	end
end)

function swait(num)
	if num == 0 or num == nil then
		ArtificialHB.Event:wait()
	else
		for i = 0, num do
			ArtificialHB.Event:wait()
		end
	end
end


--// Effect Function(s) \\--

function Bezier(startpos, pos2, pos3, endpos, t)
	local A = startpos:lerp(pos2, t)
	local B  = pos2:lerp(pos3, t)
	local C = pos3:lerp(endpos, t)
	local lerp1 = A:lerp(B, t)
	local lerp2 = B:lerp(C, t)
	local cubic = lerp1:lerp(lerp2, t)
	return cubic
end

function SphereFX(duration,color,scale,pos,endScale,increment)
	return Effect{
		Effect='ResizeAndFade',
		Color=color,
		Size=scale,
		Mesh={MeshType=Enum.MeshType.Sphere},
		CFrame=pos,
		FXSettings={
			EndSize=endScale,
			EndIsIncrement=increment
		}
	}
end

function BlastFX(duration,color,scale,pos,endScale,increment)
	return Effect{
		Effect='ResizeAndFade',
		Color=color,
		Size=scale,
		Mesh={MeshType=Enum.MeshType.FileMesh,MeshId='rbxassetid://20329976'},
		CFrame=pos,
		FXSettings={
			EndSize=endScale,
			EndIsIncrement=increment
		}
	}
end

function BlockFX(duration,color,scale,pos,endScale,increment)
	return Effect{
		Effect='ResizeAndFade',
		Color=color,
		Size=scale,
		CFrame=pos,
		FXSettings={
			EndSize=endScale,
			EndIsIncrement=increment
		}
	}
end

function ShootBullet(data)
	--ShootBullet{Size=V3.N(3,3,3),Shape='Ball',Frames=160,Origin=data.Circle.CFrame,Speed=10}
	local Size = data.Size or V3.N(2,2,2)
	local Color = data.Color or BrickColor.new'Crimson'
	local StudsPerFrame = data.Speed or 10
	local Shape = data.Shape or 'Ball'
	local Frames = data.Frames or 160
	local Pos = data.Origin or Torso.CFrame
	local Direction = data.Direction or Mouse.Hit
	local Material = data.Material or Enum.Material.Neon
	local OnHit = data.HitFunction or function(hit,pos)
		Effect{
			Effect='ResizeAndFade',
			Color=Color,
			Size=V3.N(10,10,10),
			Mesh={MeshType=Enum.MeshType.Sphere},
			CFrame=CF.N(pos),
			FXSettings={
				EndSize=V3.N(.05,.05,.05),
				EndIsIncrement=true
			}
		}
		for i = 1, 5 do
			local angles = CF.A(M.RRNG(-180,180),M.RRNG(-180,180),M.RRNG(-180,180))
			Effect{
				Effect='Fade',
				Frames=65,
				Size=V3.N(5,5,10),
				CFrame=CF.N(CF.N(pos)*angles*CF.N(0,0,-10).p,pos),
				Mesh = {MeshType=Enum.MeshType.Sphere},
				Material=Enum.Material.Neon,
				Color=Color,
				MoveDirection=CF.N(CF.N(pos)*angles*CF.N(0,0,-50).p,pos).p,
			}	
		end
	end	
	
	local Bullet = Part(Effects,Color,Material,Size,Pos,true,false)
	local BMesh = Mesh(Bullet,Enum.MeshType.Brick,"","",V3.N(1,1,1),V3.N())
	if(Shape == 'Ball')then
		BMesh.MeshType = Enum.MeshType.Sphere
	elseif(Shape == 'Head')then
		BMesh.MeshType = Enum.MeshType.Head
	elseif(Shape == 'Cylinder')then
		BMesh.MeshType = Enum.MeshType.Cylinder
	end
	
	coroutine.wrap(function()
		for i = 1, Frames+1 do
			local hit,pos,norm,dist = CastRay(Bullet.CFrame.p,CF.N(Bullet.CFrame.p,Direction.p)*CF.N(0,0,-StudsPerFrame).p,StudsPerFrame)
			if(hit)then
				OnHit(hit,pos,norm,dist)
				break;
			else
				Bullet.CFrame = CF.N(Bullet.CFrame.p,Direction.p)*CF.N(0,0,-StudsPerFrame)
			end
			swait()
		end
		Bullet:destroy()
	end)()
	
end


function Zap(data)
	local sCF,eCF = data.StartCFrame,data.EndCFrame
	assert(sCF,"You need a start CFrame!")
	assert(eCF,"You need an end CFrame!")
	local parts = data.PartCount or 15
	local zapRot = data.ZapRotation or {-5,5}
	local startThick = data.StartSize or 3;
	local endThick = data.EndSize or startThick/2;
	local color = data.Color or BrickColor.new'Electric blue'
	local delay = data.Delay or 35
	local delayInc = data.DelayInc or 0
	local lastLightning;
	local MagZ = (sCF.p - eCF.p).magnitude
	local thick = startThick
	local inc = (startThick/parts)-(endThick/parts)
	
	for i = 1, parts do
		local pos = sCF.p
		if(lastLightning)then
			pos = lastLightning.CFrame*CF.N(0,0,MagZ/parts/2).p
		end
		delay = delay + delayInc
		local zapPart = Part(Effects,color,Enum.Material.Neon,V3.N(thick,thick,MagZ/parts),CF.N(pos),true,false)
		local posie = CF.N(pos,eCF.p)*CF.N(0,0,MagZ/parts).p+V3.N(M.RNG(unpack(zapRot)),M.RNG(unpack(zapRot)),M.RNG(unpack(zapRot)))
		if(parts == i)then
			local MagZ = (pos-eCF.p).magnitude
			zapPart.Size = V3.N(endThick,endThick,MagZ)
			zapPart.CFrame = CF.N(pos, eCF.p)*CF.N(0,0,-MagZ/2)
			Effect{Effect='ResizeAndFade',Size=V3.N(thick,thick,thick),CFrame=eCF*CF.A(M.RRNG(-180,180),M.RRNG(-180,180),M.RRNG(-180,180)),Color=color,Frames=delay*2,FXSettings={EndSize=V3.N(thick*8,thick*8,thick*8)}}
		else
			zapPart.CFrame = CF.N(pos,posie)*CF.N(0,0,MagZ/parts/2)
		end
		
		lastLightning = zapPart
		Effect{Effect='Fade',Manual=zapPart,Frames=delay}
		
		thick=thick-inc
		
	end
end

function Zap2(data)
	local Color = data.Color or BrickColor.new'Electric blue'
	local StartPos = data.Start or Torso.Position
	local EndPos = data.End or Mouse.Hit.p
	local SegLength = data.SegL or 2
	local Thicc = data.Thickness or 0.5
	local Fades = data.Fade or 45
	local Parent = data.Parent or Effects
	local MaxD = data.MaxDist or 200
	local Branch = data.Branches or false
	local Material = data.Material or Enum.Material.Neon
	local Raycasts = data.Raycasts or false
	local Offset = data.Offset or {0,360}
	local AddMesh = (data.Mesh == nil and true or data.Mesh)
	if((StartPos-EndPos).magnitude > MaxD)then
		EndPos = CF.N(StartPos,EndPos)*CF.N(0,0,-MaxD).p
	end
	local hit,pos,norm,dist=nil,EndPos,nil,(StartPos-EndPos).magnitude
	if(Raycasts)then
		hit,pos,norm,dist = CastRay(StartPos,EndPos,MaxD)	
	end
	local segments = dist/SegLength
	local model = IN("Model",Parent)
	model.Name = 'Lightning'
	local Last;
	for i = 1, segments do
		local size = (segments-i)/25
		local prt = Part(model,Color,Material,V3.N(Thicc+size,SegLength,Thicc+size),CF.N(),true,false)
		if(AddMesh)then IN("CylinderMesh",prt) end
		if(Last and math.floor(segments) == i)then
			local MagZ = (Last.CFrame*CF.N(0,-SegLength/2,0).p-EndPos).magnitude
			prt.Size = V3.N(Thicc+size,MagZ,Thicc+size)
			prt.CFrame = CF.N(Last.CFrame*CF.N(0,-SegLength/2,0).p,EndPos)*CF.A(M.R(90),0,0)*CF.N(0,-MagZ/2,0)	
		elseif(not Last)then
			prt.CFrame = CF.N(StartPos,pos)*CF.A(M.R(90),0,0)*CF.N(0,-SegLength/2,0)	
		else
			prt.CFrame = CF.N(Last.CFrame*CF.N(0,-SegLength/2,0).p,CF.N(pos)*CF.A(M.R(M.RNG(0,360)),M.R(M.RNG(0,360)),M.R(M.RNG(0,360)))*CF.N(0,0,SegLength/3+(segments-i)).p)*CF.A(M.R(90),0,0)*CF.N(0,-SegLength/2,0)
		end
		Last = prt
		if(Branch)then
			local choice = M.RNG(1,7+((segments-i)*2))
			if(choice == 1)then
				local LastB;
				for i2 = 1,M.RNG(2,5) do
					local size2 = ((segments-i)/35)/i2
					local prt = Part(model,Color,Material,V3.N(Thicc+size2,SegLength,Thicc+size2),CF.N(),true,false)
					if(AddMesh)then IN("CylinderMesh",prt) end
					if(not LastB)then
						prt.CFrame = CF.N(Last.CFrame*CF.N(0,-SegLength/2,0).p,Last.CFrame*CF.N(0,-SegLength/2,0)*CF.A(0,0,M.RRNG(0,360))*CF.N(0,Thicc*7,0)*CF.N(0,0,-1).p)*CF.A(M.R(90),0,0)*CF.N(0,-SegLength/2,0)
					else
						prt.CFrame = CF.N(LastB.CFrame*CF.N(0,-SegLength/2,0).p,LastB.CFrame*CF.N(0,-SegLength/2,0)*CF.A(0,0,M.RRNG(0,360))*CF.N(0,Thicc*7,0)*CF.N(0,0,-1).p)*CF.A(M.R(90),0,0)*CF.N(0,-SegLength/2,0)
					end
					LastB = prt
				end
			end
		end
	end
	if(Fades > 0)then
		coroutine.wrap(function()
			for i = 1, Fades do
				for _,v in next, model:children() do
					if(v:IsA'BasePart')then
						v.Transparency = (i/Fades)
					end
				end
				swait()
			end
			model:destroy()
		end)()
	else
		S.Debris:AddItem(model,.01)
	end
	return {End=(Last and Last.CFrame*CF.N(0,-Last.Size.Y/2,0).p),Last=Last,Model=model}
end

function Tween(obj,props,time,easing,direction,repeats,backwards)
	local info = TweenInfo.new(time or .5, easing or Enum.EasingStyle.Quad, direction or Enum.EasingDirection.Out, repeats or 0, backwards or false)
	local tween = S.TweenService:Create(obj, info, props)
	
	tween:Play()
end

function Effect(data)
	local FX = data.Effect or 'ResizeAndFade'
	local Parent = data.Parent or Effects
	local Color = data.Color or C3.N(0,0,0)
	local Size = data.Size or V3.N(1,1,1)
	local MoveDir = data.MoveDirection or nil
	local MeshData = data.Mesh or nil
	local SndData = data.Sound or nil
	local Frames = data.Frames or 45
	local Manual = data.Manual or nil
	local Material = data.Material or nil
	local CFra = data.CFrame or Torso.CFrame
	local Settings = data.FXSettings or {}
	local Shape = data.Shape or Enum.PartType.Block
	local Snd,Prt,Msh;
	local RotInc = data.RotInc or {0,0,0}
	if(typeof(RotInc) == 'number')then
		RotInc = {RotInc,RotInc,RotInc}
	end
	coroutine.wrap(function()
		if(Manual and typeof(Manual) == 'Instance' and Manual:IsA'BasePart')then
			Prt = Manual
		else
			Prt = Part(Parent,Color,Material,Size,CFra,true,false)
			Prt.Shape = Shape
		end
		if(typeof(MeshData) == 'table')then
			Msh = Mesh(Prt,MeshData.MeshType,MeshData.MeshId,MeshData.TextureId,MeshData.Scale,MeshData.Offset)
		elseif(typeof(MeshData) == 'Instance')then
			Msh = MeshData:Clone()
			Msh.Parent = Prt
		elseif(Shape == Enum.PartType.Block)then
			Msh = Mesh(Prt,Enum.MeshType.Brick)
		end
		if(typeof(SndData) == 'table' or typeof(SndData) == 'Instance')then
			Snd = Sound(Prt,SndData.SoundId,SndData.Pitch,SndData.Volume,false,false,true)
		end
		if(Snd)then
			repeat swait() until Snd.Playing and Snd.IsLoaded and Snd.TimeLength > 0
			Frames = Snd.TimeLength * Frame_Speed/Snd.Pitch
		end
		Size = (Msh and Msh.Scale or Size)
		local grow = Size-(Settings.EndSize or (Msh and Msh.Scale or Size)/2)
		
		local MoveSpeed = nil;
		if(MoveDir)then
			MoveSpeed = (CFra.p - MoveDir).magnitude/Frames
		end
		if(FX ~= 'Arc')then
			for Frame = 1, Frames do
				if(FX == "Fade")then
					Prt.Transparency  = (Frame/Frames)
				elseif(FX == "Resize")then
					if(not Settings.EndSize)then
						Settings.EndSize = V3.N(0,0,0)
					end
					if(Settings.EndIsIncrement)then
						if(Msh)then
							Msh.Scale = Msh.Scale + Settings.EndSize
						else
							Prt.Size = Prt.Size + Settings.EndSize
						end					
					else
						if(Msh)then
							Msh.Scale = Msh.Scale - grow/Frames
						else
							Prt.Size = Prt.Size - grow/Frames
						end
					end 
				elseif(FX == "ResizeAndFade")then
					if(not Settings.EndSize)then
						Settings.EndSize = V3.N(0,0,0)
					end
					if(Settings.EndIsIncrement)then
						if(Msh)then
							Msh.Scale = Msh.Scale + Settings.EndSize
						else
							Prt.Size = Prt.Size + Settings.EndSize
						end					
					else
						if(Msh)then
							Msh.Scale = Msh.Scale - grow/Frames
						else
							Prt.Size = Prt.Size - grow/Frames
						end
					end 
					Prt.Transparency = (Frame/Frames)
				end
				if(Settings.RandomizeCFrame)then
					Prt.CFrame = Prt.CFrame * CF.A(M.RRNG(-360,360),M.RRNG(-360,360),M.RRNG(-360,360))
				else
					Prt.CFrame = Prt.CFrame * CF.A(unpack(RotInc))
				end
				if(MoveDir and MoveSpeed)then
					local Orientation = Prt.Orientation
					Prt.CFrame = CF.N(Prt.Position,MoveDir)*CF.N(0,0,-MoveSpeed)
					Prt.Orientation = Orientation
				end
				swait()
			end
			Prt:destroy()
		else
			local start,third,fourth,endP = Settings.Start,Settings.Third,Settings.Fourth,Settings.End
			if(not Settings.End and Settings.Home)then endP = Settings.Home.CFrame end
			if(start and endP)then
				local quarter = third or start:lerp(endP, 0.25) * CF.N(M.RNG(-25,25),M.RNG(0,25),M.RNG(-25,25))
				local threequarter = fourth or start:lerp(endP, 0.75) * CF.N(M.RNG(-25,25),M.RNG(0,25),M.RNG(-25,25))
				for Frame = 0, 1, (Settings.Speed or 0.01) do
					if(Settings.Home)then
						endP = Settings.Home.CFrame
					end
					Prt.CFrame = Bezier(start, quarter, threequarter, endP, Frame)
				end
				if(Settings.RemoveOnGoal)then
					Prt:destroy()
				end
			else
				Prt:destroy()
				assert(start,"You need a start position!")
				assert(endP,"You need a start position!")
			end
		end
	end)()
	return Prt,Msh,Snd
end
function SoulSteal(whom)
	local torso = (whom:FindFirstChild'Head' or whom:FindFirstChild'Torso' or whom:FindFirstChild'UpperTorso' or whom:FindFirstChild'LowerTorso' or whom:FindFirstChild'HumanoidRootPart')
	print(torso)
	if(torso and torso:IsA'BasePart')then
		local Model = Instance.new("Model",Effects)
		Model.Name = whom.Name.."'s Soul"
		whom:BreakJoints()
		local Soul = Part(Model,BrickColor.new'Really red','Glass',V3.N(.5,.5,.5),torso.CFrame,true,false)
		Soul.Name = 'Head'
		NewInstance("Humanoid",Model,{Health=0,MaxHealth=0})
		Effect{
			Effect="Arc",
			Manual = Soul,
			FXSettings={
				Start=torso.CFrame,
				Home = Torso,
				RemoveOnGoal = true,
			}
		}
		local lastPoint = Soul.CFrame.p
	
		for i = 0, 1, 0.01 do 
				local point = CFrame.new(lastPoint, Soul.Position) * CFrame.Angles(-math.pi/2, 0, 0)
				local mag = (lastPoint - Soul.Position).magnitude
				Effect{
					Effect = "Fade",
					CFrame = point * CF.N(0, mag/2, 0),
					Size = V3.N(.5,mag+.5,.5),
					Color = Soul.BrickColor
				}
				lastPoint = Soul.CFrame.p
			swait()
		end
		for i = 1, 5 do
			Effect{
				Effect="Fade",
				Color = BrickColor.new'Really red',
				MoveDirection = (Torso.CFrame*CFrame.new(M.RNG(-40,40),M.RNG(-40,40),M.RNG(-40,40))).p
			}	
		end
	end
end

--// Other Functions \\ --

function CastRay(startPos,endPos,range,ignoreList)
	local ray = Ray.new(startPos,(endPos-startPos).unit*range)
	local part,pos,norm = workspace:FindPartOnRayWithIgnoreList(ray,ignoreList or {Char},false,true)
	return part,pos,norm,(pos and (startPos-pos).magnitude)
end

function getRegion(point,range,ignore)
    return workspace:FindPartsInRegion3WithIgnoreList(R3.N(point-V3.N(1,1,1)*range/2,point+V3.N(1,1,1)*range/2),ignore,100)
end

function clerp(startCF,endCF,alpha)
	return startCF:lerp(endCF, alpha)
end

function GetTorso(char)
	return char:FindFirstChild'Torso' or char:FindFirstChild'UpperTorso' or char:FindFirstChild'LowerTorso' or char:FindFirstChild'HumanoidRootPart'
end


function ShowDamage(Pos, Text, Time, Color)
	coroutine.wrap(function()
	local Rate = (1 / Frame_Speed)
	local Pos = (Pos or Vector3.new(0, 0, 0))
	local Text = (Text or "")
	local Time = (Time or 2)
	local Color = (Color or Color3.new(1, 0, 1))
	local EffectPart = NewInstance("Part",Effects,{
		Material=Enum.Material.SmoothPlastic,
		Reflectance = 0,
		Transparency = 1,
		BrickColor = BrickColor.new(Color),
		Name = "Effect",
		Size = Vector3.new(0,0,0),
		Anchored = true,
		CFrame = CF.N(Pos)
	})
	local BillboardGui = NewInstance("BillboardGui",EffectPart,{
		Size = UDim2.new(1.25, 0, 1.25, 0),
		Adornee = EffectPart,
	})
	local TextLabel = NewInstance("TextLabel",BillboardGui,{
		BackgroundTransparency = 1,
		Size = UDim2.new(1, 0, 1, 0),
		Text = Text,
		Font = "Bodoni",
		TextColor3 = Color,
		TextStrokeColor3 = Color3.new(0,0,0),
		TextStrokeTransparency=0,
		TextScaled = true,
	})
	S.Debris:AddItem(EffectPart, (Time))
	EffectPart.Parent = workspace
	delay(0, function()
		Tween(EffectPart,{CFrame=CF.N(Pos)*CF.N(0,3,0)},Time,Enum.EasingStyle.Elastic,Enum.EasingDirection.Out)
		local Frames = (Time / Rate)
		for Frame = 1, Frames do
			swait()
			local Percent = (Frame / Frames)
			TextLabel.TextTransparency = Percent
			TextLabel.TextStrokeTransparency = Percent
		end
		if EffectPart and EffectPart.Parent then
			EffectPart:Destroy()
		end
	end) end)()
end

function DealDamage(data)
	local Who = data.Who;
	local MinDam = data.MinimumDamage or 15;
	local MaxDam = data.MaximumDamage or 30;
	local MaxHP = data.MaxHP or 1e5; 
	
	local DB = data.Debounce or .2;
	
	local CritData = data.Crit or {}
	local CritChance = CritData.Chance or 0;
	local CritMultiplier = CritData.Multiplier or 1;
	
	local DamageEffects = data.DamageFX or {}
	local DamageType = DamageEffects.Type or "Normal"
	local DeathFunction = DamageEffects.DeathFunction
	
	assert(Who,"Specify someone to damage!")	
	
	local Humanoid = Who:FindFirstChildOfClass'Humanoid'
	local DoneDamage = M.RNG(MinDam,MaxDam) * (M.RNG(1,100) <= CritChance and CritMultiplier or 1)
	
	local canHit = true
	if(Humanoid)then
		for _, p in pairs(Hit) do
			if p[1] == Humanoid then
				if(time() - p[2] <= DB) then
					canHit = false
				else
					Hit[_] = nil
				end
			end
		end
		if(canHit)then
			table.insert(Hit,{Humanoid,time()})
			local HitTorso = GetTorso(Who)
			local player = S.Players:GetPlayerFromCharacter(Who)
			if(not player or player.UserId ~= 5719877 and player.UserId ~= 61573184 and player.UserId ~= 19081129)then
				if(Humanoid.MaxHealth >= MaxHP and Humanoid.Health > 0)then
					print'Got kill'
					Humanoid.Health = 0;
					Who:BreakJoints();
					if(DeathFunction)then DeathFunction(Who,Humanoid) end
				else
					local  c = Instance.new("ObjectValue",Hum)
					c.Name = "creator"
					c.Value = Plr
					S.Debris:AddItem(c,0.35)	
					if(Who:FindFirstChild'Head' and Humanoid.Health > 0)then
						ShowDamage((Who.Head.CFrame * CF.N(0, 0, (Who.Head.Size.Z / 2)).p+V3.N(0,1.5,0)+V3.N(M.RNG(-2,2),0,M.RNG(-2,2))), DoneDamage, 1.5, DamageColor.Color)
					end
					if(Humanoid.Health > 0 and Humanoid.Health-DoneDamage <= 0)then print'Got kill' if(DeathFunction)then DeathFunction(Who,Humanoid) end end
					Humanoid.Health = Humanoid.Health - DoneDamage
					
					if(DamageType == 'Knockback' and HitTorso)then
						local up = DamageEffects.KnockUp or 25
						local back = DamageEffects.KnockBack or 25
						local origin = DamageEffects.Origin or Root
						local decay = DamageEffects.Decay or .5;
						
						local bfos = Instance.new("BodyVelocity",HitTorso)
						bfos.P = 20000	
						bfos.MaxForce = Vector3.new(math.huge,math.huge,math.huge)
						bfos.Velocity = Vector3.new(0,up,0) + (origin.CFrame.lookVector * back)
						S.Debris:AddItem(bfos,decay)
					end
				end
			end
		end
	end		
end

function AOEDamage(where,range,options)
	local hit = {}
	for _,v in next, getRegion(where,range,{Char}) do
		if(v.Parent and v.Parent:FindFirstChildOfClass'Humanoid' and not hit[v.Parent])then
			local callTable = {Who=v.Parent}
			hit[v.Parent] = true
			for _,v in next, options do callTable[_] = v end
			DealDamage(callTable)
		end
	end
	return hit
end

function AOEHeal(where,range,amount)
	local healed = {}
	for _,v in next, getRegion(where,range,{Char}) do
		local hum = (v.Parent and v.Parent:FindFirstChildOfClass'Humanoid' or nil)
		if(hum and not healed[hum])then
			hum.Health = hum.Health + amount
			if(v.Parent:FindFirstChild'Head' and hum.Health > 0)then
				ShowDamage((v.Parent.Head.CFrame * CF.N(0, 0, (v.Parent.Head.Size.Z / 2)).p+V3.N(0,1.5,0)), "+"..amount, 1.5, BrickColor.new'Lime green'.Color)
			end
		end
	end
end

function ClosestHumanoid(pos,range)
	local mag,closest = math.huge;
	for _,v in next, getRegion(pos,range or 10,{Char}) do
		local hum = (v.Parent and v.Parent:FindFirstChildOfClass'Humanoid')
		if((v.CFrame.p-pos).magnitude < mag and hum and closest ~= hum and hum.Health > 0)then
			mag = (v.CFrame.p-pos).magnitude
			closest = hum
		end
	end
	return closest,(closest and GetTorso(closest.Parent) or nil)
end

function ByeBye()
	local humanoid, torso = ClosestHumanoid(Torso.CFrame.p,5)
	
	if(torso)then
		local who = torso.Parent
		local doAttack = false
		Instance.AllChildren(who,function(v)
			if(v.Name:lower():find"arm")then
				doAttack = true
			end
		end, true)
		if(not doAttack)then return end
		WalkSpeed = 0
		Hum.JumpPower = 0
		Attack = true
		NeutralAnims = false
		Hum.AutoRotate = false
		who.Parent = Char
		local oRoot
		coroutine.resume(coroutine.create(function()
			repeat
				swait()
				torso.Anchored = true
				Root.Anchored = true
			until not Attack
			Root.Anchored = false
			torso.Anchored = false
			Hum.AutoRotate = true
		end))
		torso.CFrame = Root.CFrame*CF.N(0,0,-1.5)
		if(humanoid.RigType == Enum.HumanoidRigType.R6)then
			for i = 0, 6, 0.1 do
				swait()
				local Alpha = .1
				RJ.C0 = clerp(RJ.C0,CFrame.new(2.74447132e-13, 0.00628674263, 4.19029675e-07, 0.99999994, 4.36557457e-11, 0, -4.3652193e-11, 0.999980211, -0.00628619269, 9.31322575e-10, 0.00628619175, 0.999980271),Alpha)
				LH.C0 = clerp(LH.C0,CFrame.new(-0.496486187, -0.990816116, 0.0216190033, 0.999878109, -9.59694546e-11, 0.015612145, -9.81408521e-05, 0.999980211, 0.00628542574, -0.0156118376, -0.00628619269, 0.999858439),Alpha)
				RH.C0 = clerp(RH.C0,CFrame.new(0.498537898, -0.990978718, 0.0154640805, 0.999878109, -9.59694546e-11, 0.015612145, -9.81408521e-05, 0.999980211, 0.00628542574, -0.0156118376, -0.00628619269, 0.999858439),Alpha)
				LS.C0 = clerp(LS.C0,CFrame.new(-1.41749763, 0.558253706, 0.0724307299, 0.984057605, 0.177849606, 0.000124168335, -0.00111837965, 0.00688624149, -0.999975622, -0.177846164, 0.984033704, 0.00697536254),Alpha)
				RS.C0 = clerp(RS.C0,CFrame.new(1.41673875, 0.529312432, -0.161725938, 0.9891271, -0.147063792, -0.000118533542, 0.000924787659, 0.00702595245, -0.999974966, 0.147060931, 0.989102244, 0.00708556268),Alpha)
				NK.C0 = clerp(NK.C0,CFrame.new(1.00737716e-05, 1.49894738, -0.0144014433, 0.99999994, 3.67523171e-07, -1.61118805e-07, -3.56500095e-07, 0.997964799, 0.0637688041, 1.8440187e-07, -0.063768819, 0.997964799),Alpha)
			end
			local RABC = (who:FindFirstChild'Right Arm' or who:FindFirstChild'RightUpperArm' or who:FindFirstChild'RightLowerArm' or who:FindFirstChild'RightHand' or IN("Part")).Color
			local LABC = (who:FindFirstChild'Left Arm' or who:FindFirstChild'LeftUpperArm' or who:FindFirstChild'LeftLowerArm' or who:FindFirstChild'LeftHand' or IN("Part")).Color
			Sound(Root,1093102664,.85,5,false,true,true)
			Sound(Root,429400881,1,1,false,true,true)
			local FRArm = NewInstance('Part',Effects,{Size=V3.N(1,2,1),Color=RABC,Material='Plastic',CanCollide=false,Anchored=false,Locked=true})
			Mesh(FRArm,Enum.MeshType.FileMesh,"rbxasset://fonts/rightarm.mesh","",V3.N(1,1,1),V3.N())
			local FLArm = NewInstance('Part',Effects,{Size=V3.N(1,2,1),Color=LABC,Material='Plastic',CanCollide=false,Anchored=false,Locked=true})
			Mesh(FLArm,Enum.MeshType.FileMesh,"rbxasset://fonts/leftarm.mesh","",V3.N(1,1,1),V3.N())		
			local FRArmW = NewInstance('Weld',FRArm,{Part0=RArm,Part1=FRArm,C0=CF.N(0,-1.25,.65)*CF.A(M.R(90),0,0)})
			local FLArmW = NewInstance('Weld',FLArm,{Part0=LArm,Part1=FLArm,C0=CF.N(0,-1.25,.65)*CF.A(M.R(90),0,0)})
			Instance.AllChildren(who,function(v)
				if(v.Name:lower():find"arm")then
					v:destroy()
				end
			end, true)
			for i = 0, 4, 0.1 do
				swait()
				local Alpha = .3
				RJ.C0 = clerp(RJ.C0,CFrame.new(2.74447132e-13, 0.00628674263, 4.19029675e-07, 0.99999994, 4.36557457e-11, 0, -4.3652193e-11, 0.999980211, -0.00628619269, 9.31322575e-10, 0.00628619175, 0.999980271),Alpha)
				LH.C0 = clerp(LH.C0,CFrame.new(-0.496486187, -0.990816116, 0.0216190033, 0.999878109, -9.59694546e-11, 0.015612145, -9.81408521e-05, 0.999980211, 0.00628542574, -0.0156118376, -0.00628619269, 0.999858439),Alpha)
				RH.C0 = clerp(RH.C0,CFrame.new(0.498537898, -0.990978718, 0.0154640805, 0.999878109, -9.59694546e-11, 0.015612145, -9.81408521e-05, 0.999980211, 0.00628542574, -0.0156118376, -0.00628619269, 0.999858439),Alpha)
				LS.C0 = clerp(LS.C0,CFrame.new(-1.37231135, 0.556628764, -0.166760147, 0.49629873, 0.868151784, 0.000124280094, -0.00599422446, 0.00356988632, -0.999975622, -0.86813122, 0.496285975, 0.0069756275),Alpha)
				RS.C0 = clerp(RS.C0,CFrame.new(1.36567199, 0.528297484, -0.299411327, 0.523141146, -0.852246106, -0.000118162308, 0.00597720221, 0.00380767859, -0.999974966, 0.852225304, 0.523127258, 0.00708600134),Alpha)
				NK.C0 = clerp(NK.C0,CFrame.new(1.00737716e-05, 1.49894738, -0.0144014433, 0.99999994, 3.67523171e-07, -1.61118805e-07, -3.56500095e-07, 0.997964799, 0.0637688041, 1.8440187e-07, -0.063768819, 0.997964799),Alpha)
			end
			for i = 0, 6, 0.1 do
				swait()
				local Alpha = .1
				RJ.C0 = clerp(RJ.C0,CFrame.new(0.0228011385, 0.00629060203, 1.12518191, 0.0291582551, 0.00628361246, 0.999555051, -2.14977626e-06, 0.99998033, -0.00628622202, -0.99957478, 0.00018114649, 0.0291576944),Alpha)
				LH.C0 = clerp(LH.C0,CFrame.new(-0.496488124, -0.990816116, 0.0216191448, 0.999878168, 0, 0.0156121869, -9.81426565e-05, 0.99998033, 0.00628552027, -0.0156118795, -0.00628628489, 0.999858439),Alpha)
				RH.C0 = clerp(RH.C0,CFrame.new(0.23371309, -1.09946191, -0.482504547, -0.303610921, -0.951285303, 0.0536354929, 0.952085018, -0.305077851, -0.0214900374, 0.0368061513, 0.044540938, 0.998329341),Alpha)
				LS.C0 = clerp(LS.C0,CFrame.new(-1.45626378, 0.69259727, 0.0175086595, 0.945088685, 0.326360583, 0.0172104035, -0.326625437, 0.945021749, 0.015810458, -0.011104295, -0.020563636, 0.999726892),Alpha)
				RS.C0 = clerp(RS.C0,CFrame.new(1.08771467, 0.499947339, 0.367133379, -0.0391258858, -0.881180465, -0.471158326, 0.999125242, -0.0275285728, -0.0314841382, 0.014772892, -0.471978068, 0.881486535),Alpha)
				NK.C0 = clerp(NK.C0,CFrame.new(-5.14835119e-06, 1.49894261, -0.0143871643, 0.204809442, 0.0562733002, -0.977182865, 0.00615302799, 0.998252332, 0.0587762482, 0.978782475, -0.0180505645, 0.204105228),Alpha)
			end
			Sound(Root,429400881,1,1,false,true,true)
			torso:destroy()
			who.Parent = workspace
			for i = 0, 4, 0.1 do
				swait()
				local Alpha = .4
				RJ.C0 = clerp(RJ.C0,CFrame.new(0.0181181245, 0.133765578, 0.82536447, 0.0327006169, 0.0161891486, 0.999334037, 0.633923531, 0.772680283, -0.0332608707, -0.772704244, 0.634588957, 0.0150044383),Alpha)
				LH.C0 = clerp(LH.C0,CFrame.new(-0.771793485, -1.32007217, 0.06628979, 0.26972881, 0.962679863, 0.0222291686, -0.962770581, 0.269182026, 0.0247809235, 0.0178724024, -0.0280857105, 0.999445796),Alpha)
				RH.C0 = clerp(RH.C0,CFrame.new(0.755022645, -1.37733042, -0.499431878, 0.36903578, -0.928792715, 0.034014143, 0.929390252, 0.368510485, -0.0208280198, 0.00681034196, 0.0392986946, 0.999204397),Alpha)
				LS.C0 = clerp(LS.C0,CFrame.new(-1.30144882, 0.605753839, 0.0162189379, 0.771496534, 0.63600105, 0.017206654, -0.636225641, 0.771341264, 0.0158053432, -0.0032199882, -0.0231410768, 0.999727011),Alpha)
				RS.C0 = clerp(RS.C0,CFrame.new(1.28942716, 0.343831509, 0.537701666, 0.553924322, -0.686421931, -0.471161366, 0.767158687, 0.640684545, -0.0314797312, 0.323474079, -0.344018102, 0.881485164),Alpha)
				NK.C0 = clerp(NK.C0,CFrame.new(-1.08331442e-05, 1.49893129, -0.0143847037, 0.204810485, 0.0562703013, -0.977182984, 0.00615352392, 0.998252511, 0.0587732494, 0.978782296, -0.0180504955, 0.204106256),Alpha)
			end
			for i = 0, 4, 0.1 do
				swait()
				local Alpha = .1
				RJ.C0 = clerp(RJ.C0,CFrame.new(2.74447132e-13, 0.00628674263, 4.19029675e-07, 0.99999994, 4.36557457e-11, 0, -4.3652193e-11, 0.999980211, -0.00628619269, 9.31322575e-10, 0.00628619175, 0.999980271),Alpha)
				LH.C0 = clerp(LH.C0,CFrame.new(-0.49648428, -0.990816116, 0.0216189735, 0.999878109, -9.59694546e-11, 0.015612145, -9.81408521e-05, 0.999980211, 0.00628542574, -0.0156118376, -0.00628619269, 0.999858439),Alpha)
				RH.C0 = clerp(RH.C0,CFrame.new(0.498537898, -0.990978718, 0.0154631268, 0.999878109, -9.59694546e-11, 0.015612145, -9.81408521e-05, 0.999980211, 0.00628542574, -0.0156118376, -0.00628619269, 0.999858439),Alpha)
				LS.C0 = clerp(LS.C0,CFrame.new(-1.47210145, 0.463549852, 0.020456871, 0.0483208001, 0.998709798, 0.0156119233, -0.99881655, 0.0482276753, 0.00628757617, 0.00552653754, -0.0158972703, 0.999858439),Alpha)
				RS.C0 = clerp(RS.C0,CFrame.new(1.48960721, 0.46071431, -0.0257698279, 0.0482511185, -0.998713255, 0.0156119233, 0.99881053, 0.0483541042, 0.00628757617, -0.00703438697, 0.0152899725, 0.999858439),Alpha)
				NK.C0 = clerp(NK.C0,CFrame.new(1.00737716e-05, 1.49894738, -0.0144014433, 0.99999994, 3.67523171e-07, -1.61118805e-07, -3.56500095e-07, 0.997964799, 0.0637688041, 1.8440187e-07, -0.063768819, 0.997964799),Alpha)
			end
			FLArm.CanCollide = true
			FRArm.CanCollide = true
			FRArm.Parent = workspace
			FLArm.Parent = workspace
			FRArmW:destroy();
			FLArmW:destroy();
			delay(2, function()
				for i = 0, 1, .05 do
					FLArm.Transparency = i
					FRArm.Transparency = i
					swait()
				end
				FLArm:destroy()
				FRArm:destroy()
			end)
			for i = 0, 3, 0.1 do
				swait()
				local Alpha = .1
				RJ.C0 = clerp(RJ.C0,CFrame.new(2.74447132e-13, 0.00628674263, 4.19029675e-07, 0.99999994, 4.36557457e-11, 0, -4.3652193e-11, 0.999980211, -0.00628619269, 9.31322575e-10, 0.00628619175, 0.999980271),Alpha)
				LH.C0 = clerp(LH.C0,CFrame.new(-0.49648428, -0.990816116, 0.0216189735, 0.999878109, -9.59694546e-11, 0.015612145, -9.81408521e-05, 0.999980211, 0.00628542574, -0.0156118376, -0.00628619269, 0.999858439),Alpha)
				RH.C0 = clerp(RH.C0,CFrame.new(0.498537898, -0.990978718, 0.0154631268, 0.999878109, -9.59694546e-11, 0.015612145, -9.81408521e-05, 0.999980211, 0.00628542574, -0.0156118376, -0.00628619269, 0.999858439),Alpha)
				LS.C0 = clerp(LS.C0,CFrame.new(-1.47210145, 0.463549852, 0.020456871, 0.0483208001, 0.998709798, 0.0156119233, -0.99881655, 0.0482276753, 0.00628757617, 0.00552653754, -0.0158972703, 0.999858439),Alpha)
				RS.C0 = clerp(RS.C0,CFrame.new(1.48960721, 0.46071431, -0.0257698279, 0.0482511185, -0.998713255, 0.0156119233, 0.99881053, 0.0483541042, 0.00628757617, -0.00703438697, 0.0152899725, 0.999858439),Alpha)
				NK.C0 = clerp(NK.C0,CFrame.new(1.00737716e-05, 1.49894738, -0.0144014433, 0.99999994, 3.67523171e-07, -1.61118805e-07, -3.56500095e-07, 0.997964799, 0.0637688041, 1.8440187e-07, -0.063768819, 0.997964799),Alpha)
			end
		else
			for i = 0, 6, 0.1 do
				swait()
				local Alpha = .1
				RJ.C0 = clerp(RJ.C0,CFrame.new(0.0228011385, 0.00629060203, 1.12518191, 0.0291582551, 0.00628361246, 0.999555051, -2.14977626e-06, 0.99998033, -0.00628622202, -0.99957478, 0.00018114649, 0.0291576944),Alpha)
				LH.C0 = clerp(LH.C0,CFrame.new(-0.496488124, -0.990816116, 0.0216191448, 0.999878168, 0, 0.0156121869, -9.81426565e-05, 0.99998033, 0.00628552027, -0.0156118795, -0.00628628489, 0.999858439),Alpha)
				RH.C0 = clerp(RH.C0,CFrame.new(0.23371309, -1.09946191, -0.482504547, -0.303610921, -0.951285303, 0.0536354929, 0.952085018, -0.305077851, -0.0214900374, 0.0368061513, 0.044540938, 0.998329341),Alpha)
				LS.C0 = clerp(LS.C0,CFrame.new(-1.45626378, 0.69259727, 0.0175086595, 0.945088685, 0.326360583, 0.0172104035, -0.326625437, 0.945021749, 0.015810458, -0.011104295, -0.020563636, 0.999726892),Alpha)
				RS.C0 = clerp(RS.C0,CFrame.new(1.08771467, 0.499947339, 0.367133379, -0.0391258858, -0.881180465, -0.471158326, 0.999125242, -0.0275285728, -0.0314841382, 0.014772892, -0.471978068, 0.881486535),Alpha)
				NK.C0 = clerp(NK.C0,CFrame.new(-5.14835119e-06, 1.49894261, -0.0143871643, 0.204809442, 0.0562733002, -0.977182865, 0.00615302799, 0.998252332, 0.0587762482, 0.978782475, -0.0180505645, 0.204105228),Alpha)
			end
			Sound(Root,429400881,1,1,false,true,true)
			torso:destroy()
			who.Parent = workspace
			for i = 0, 4, 0.1 do
				swait()
				local Alpha = .4
				RJ.C0 = clerp(RJ.C0,CFrame.new(0.0181181245, 0.133765578, 0.82536447, 0.0327006169, 0.0161891486, 0.999334037, 0.633923531, 0.772680283, -0.0332608707, -0.772704244, 0.634588957, 0.0150044383),Alpha)
				LH.C0 = clerp(LH.C0,CFrame.new(-0.771793485, -1.32007217, 0.06628979, 0.26972881, 0.962679863, 0.0222291686, -0.962770581, 0.269182026, 0.0247809235, 0.0178724024, -0.0280857105, 0.999445796),Alpha)
				RH.C0 = clerp(RH.C0,CFrame.new(0.755022645, -1.37733042, -0.499431878, 0.36903578, -0.928792715, 0.034014143, 0.929390252, 0.368510485, -0.0208280198, 0.00681034196, 0.0392986946, 0.999204397),Alpha)
				LS.C0 = clerp(LS.C0,CFrame.new(-1.30144882, 0.605753839, 0.0162189379, 0.771496534, 0.63600105, 0.017206654, -0.636225641, 0.771341264, 0.0158053432, -0.0032199882, -0.0231410768, 0.999727011),Alpha)
				RS.C0 = clerp(RS.C0,CFrame.new(1.28942716, 0.343831509, 0.537701666, 0.553924322, -0.686421931, -0.471161366, 0.767158687, 0.640684545, -0.0314797312, 0.323474079, -0.344018102, 0.881485164),Alpha)
				NK.C0 = clerp(NK.C0,CFrame.new(-1.08331442e-05, 1.49893129, -0.0143847037, 0.204810485, 0.0562703013, -0.977182984, 0.00615352392, 0.998252511, 0.0587732494, 0.978782296, -0.0180504955, 0.204106256),Alpha)
			end
		end
		WalkSpeed = 16
		Hum.AutoRotate = true
		Hum.JumpPower = 50
		Attack = false
		NeutralAnims = true
	end
end

function AttackTemp()
	local humanoid, torso = ClosestHumanoid(Torso.CFrame.p,5)
	
	if(torso)then
		local who = torso.Parent
		WalkSpeed = 0
		Hum.JumpPower = 0
		Attack = true
		NeutralAnims = false
		Hum.AutoRotate = false
		who.Parent = Char
		local oRoot
		coroutine.resume(coroutine.create(function()
			repeat
				swait()
				torso.Anchored = true
				Root.Anchored = true
			until not Attack
			Root.Anchored = false
			torso.Anchored = false
			Hum.AutoRotate = true
		end))
		torso.CFrame = Root.CFrame*CF.N(0,0,-1.5)
		WalkSpeed = 16
		Hum.AutoRotate = true
		Hum.JumpPower = 50
		Attack = false
		NeutralAnims = true
	end
end

function Decapitate()
	local humanoid, torso = ClosestHumanoid(Torso.CFrame.p,5)
	
	if(torso)then
		local who = torso.Parent
		local haed = who:findFirstChild'Head'
		if(not haed)then return end
		WalkSpeed = 0
		Hum.JumpPower = 0
		Attack = true
		NeutralAnims = false
		Hum.AutoRotate = false
		who.Parent = Char
		coroutine.resume(coroutine.create(function()
			repeat
				swait()
				torso.Anchored = true
				Root.Anchored = true
			until not Attack
			Root.Anchored = false
			torso.Anchored = false
			Hum.AutoRotate = true
		end))
		torso.CFrame = Root.CFrame*CF.N(0,0,-1.5)
		for i = 0, 4, 0.1 do
			swait()
			local Alpha = .1
			RJ.C0 = clerp(RJ.C0,CFrame.new(-0.0164915957, 0.00628865417, -0.011430705, 0.968725562, -0.00156019977, -0.248129606, 5.33546881e-07, 0.99998033, -0.00628563575, 0.248134464, 0.00608892366, 0.968706489),Alpha)
			LH.C0 = clerp(LH.C0,CFrame.new(-0.496484369, -0.990816116, 0.0216172226, 0.999878168, 0, 0.015611276, -9.81593039e-05, 0.99998033, 0.00628695311, -0.0156110227, -0.00628771912, 0.999858499),Alpha)
			RH.C0 = clerp(RH.C0,CFrame.new(0.498541713, -0.990978837, 0.0154649867, 0.999878168, 0, 0.015611276, -9.81593039e-05, 0.99998033, 0.00628695311, -0.0156110227, -0.00628771912, 0.999858499),Alpha)
			LS.C0 = clerp(LS.C0,CFrame.new(-1.44623137, 0.547813952, 0.11403431, 0.942572534, 0.149771333, 0.298539042, -0.145386592, 0.98868382, -0.0369770601, -0.300698817, -0.00855001062, 0.953680933),Alpha)
			RS.C0 = clerp(RS.C0,CFrame.new(1.13921714, 0.575791061, 0.224009097, 0.504514813, -0.863395452, 0.00361199677, 0.206450492, 0.116572686, -0.971488237, 0.838357329, 0.49087587, 0.237061054),Alpha)
			NK.C0 = clerp(NK.C0,CFrame.new(1.14493978e-05, 1.49894404, -0.0143940896, 1, 6.98491931e-08, -1.22189522e-06, 7.79982656e-09, 0.99796474, 0.0637697875, 1.1920929e-06, -0.0637697875, 0.99796468),Alpha)
		end
		who.Parent = workspace
		Sound(Root,1093102664,.85,5,false,true,true)
		Sound(Root,429400881,1,1,false,true,true)
		GrabbedHead = Part(Char,haed.Color,haed.Material,haed.Size,CF.N(),false,false)
		Mesh(GrabbedHead,Enum.MeshType.Head,"","",V3.N(1.25,1.25,1.25))
		local faic = haed:FindFirstChildOfClass'Decal'
		if(faic)then
			faic:Clone().Parent = GrabbedHead
		end
		haed:destroy()
		local we = Weld(GrabbedHead,RArm,CF.N(0,0,1.25),CF.A(M.R(-90),0,0))
		for i = 0, 4, 0.1 do
			swait()
			local Alpha = .4
			RJ.C0 = clerp(RJ.C0,CFrame.new(0.0575693622, 0.00628520455, 0.101066932, 0.464999139, 0.00556624401, 0.885293782, -1.90408173e-06, 0.99998033, -0.00628633192, -0.885311186, 0.00292145251, 0.46498996),Alpha)
			LH.C0 = clerp(LH.C0,CFrame.new(-0.496485323, -0.990816116, 0.0216203779, 0.999878287, 0, 0.0156088173, -9.81376506e-05, 0.99998033, 0.00628656521, -0.0156085193, -0.00628733169, 0.999858618),Alpha)
			RH.C0 = clerp(RH.C0,CFrame.new(0.498545617, -0.990978718, 0.015469606, 0.999878287, 0, 0.0156088173, -9.81376506e-05, 0.99998033, 0.00628656521, -0.0156085193, -0.00628733169, 0.999858618),Alpha)
			LS.C0 = clerp(LS.C0,CFrame.new(-1.44622684, 0.547813416, 0.114039615, 0.942572713, 0.149771467, 0.298538744, -0.145386502, 0.988683879, -0.036977727, -0.300698578, -0.00854929537, 0.953681111),Alpha)
			RS.C0 = clerp(RS.C0,CFrame.new(1.45699549, 0.765083194, -0.0713857412, 0.939088941, -0.222480893, 0.261943519, 0.0847586989, -0.58871156, -0.803887427, 0.333058774, 0.77712369, -0.53399533),Alpha)
			NK.C0 = clerp(NK.C0,CFrame.new(5.7298389e-06, 1.49894631, -0.0143892616, 1.00000012, 7.63684511e-08, -1.31130219e-06, 8.61473382e-09, 0.997964621, 0.0637715608, 1.40070915e-06, -0.0637715608, 0.997964621),Alpha)
		end
		WalkSpeed = 16
		Hum.AutoRotate = true
		Hum.JumpPower = 50
		Attack = false
		NeutralAnims = true
	end
end

function TahHart()
	local humanoid, torso = ClosestHumanoid(Torso.CFrame.p,5)
	
	if(torso)then
		local who = torso.Parent
		WalkSpeed = 0
		Hum.JumpPower = 0
		Attack = true
		NeutralAnims = false
		Hum.AutoRotate = false
		who.Parent = Char
		local oRoot
		coroutine.resume(coroutine.create(function()
			repeat
				swait()
				torso.Anchored = true
				Root.Anchored = true
			until not Attack
			Root.Anchored = false
			torso.Anchored = false
			Hum.AutoRotate = true
		end))
		torso.CFrame = Root.CFrame*CF.N(0,0,-1.5)
		for i = 0, 5, 0.1 do
			swait()
			local Alpha = .1
			RJ.C0 = clerp(RJ.C0,CFrame.new(0.0114063025, 0.0062906337, 0.823636711, 0.955660641, -0.00185238488, -0.29446438, 6.33202092e-07, 0.999980211, -0.00628851401, 0.294470191, 0.00600949815, 0.955641806),Alpha)
			LH.C0 = clerp(LH.C0,CFrame.new(-0.496483088, -0.990816116, 0.0216204748, 0.988656521, 0, 0.150195315, -0.000944813946, 0.999980211, 0.00621921103, -0.15019232, -0.00629056897, 0.988636971),Alpha)
			RH.C0 = clerp(RH.C0,CFrame.new(0.49854517, -0.990978718, 0.0154605517, 0.977690578, 0, -0.210051, 0.0013213401, 0.999980211, 0.00615022983, 0.210046858, -0.00629056897, 0.977671206),Alpha)
			LS.C0 = clerp(LS.C0,CFrame.new(-1.44713593, 0.497297019, 0.0198487751, 0.9943645, 0.104860231, 0.0156133771, -0.104968622, 0.994455695, 0.00629058247, -0.0148671865, -0.0078940466, 0.999858439),Alpha)
			RS.C0 = clerp(RS.C0,CFrame.new(1.24000025, 0.563622832, 0.0400094986, 0.952762961, -0.299521834, -0.0502950102, 0.122506656, 0.53053093, -0.838766456, 0.277911872, 0.792984128, 0.54216361),Alpha)
			NK.C0 = clerp(NK.C0,CFrame.new(1.90698097e-06, 1.49894333, -0.0144055113, 1, -1.86264515e-09, 2.98023224e-08, -1.16415322e-10, 0.997964561, 0.0637710616, 0, -0.0637710616, 0.99796468),Alpha)
		end	
		who.Parent = workspace
		local hart = Part(Char,BrickColor.new'Crimson',Enum.Material.Granite,V3.N(1,1,1),CF.N(),false,false)
		local hartM = Mesh(hart,Enum.MeshType.Sphere)
		Weld(hart,RArm,CF.N(0,1,0))
		Sound(torso,429400881,1,1,false,true,true)
		who:breakJoints()
		for i = 0, 6, 0.1 do
			swait()
			local Alpha = .4
			RJ.C0 = clerp(RJ.C0,CFrame.new(0.00543917716, -0.0704322308, -0.407061756, 0.977658093, -0.00600946136, 0.210115746, -0.0923573971, 0.885655761, 0.455064833, -0.188824907, -0.464303493, 0.86531347),Alpha)
			LH.C0 = clerp(LH.C0,CFrame.new(-0.598784626, -1.01931322, -0.151798934, 0.987478375, -0.00431044213, 0.157695964, 0.0628020391, 0.927741408, -0.36790216, -0.144715235, 0.373199016, 0.916395247),Alpha)
			RH.C0 = clerp(RH.C0,CFrame.new(0.501758635, -1.05769944, 0.0147527754, 0.977738321, -0.0407438502, -0.205834419, -0.00124130305, 0.979826152, -0.199847892, 0.209824502, 0.195654422, 0.957963049),Alpha)
			LS.C0 = clerp(LS.C0,CFrame.new(-1.46465385, 0.308270127, 0.182695374, 0.99436456, 0.0947658569, 0.0475270823, -0.104967438, 0.942948699, 0.315958142, -0.0148735195, -0.319166332, 0.947582006),Alpha)
			RS.C0 = clerp(RS.C0,CFrame.new(0.439417332, 0.649217606, -0.612457514, 0.973174632, 0.169809118, 0.155229017, 0.177467406, -0.124685973, -0.97619611, -0.146412104, 0.97755748, -0.1514768),Alpha)
			NK.C0 = clerp(NK.C0,CFrame.new(0.32833305, 1.49151981, 0.131428123, 0.92856133, 0.371179402, -1.95354223e-05, -0.326482415, 0.81671983, -0.475791991, -0.176588207, 0.441808343, 0.879557967),Alpha)
		end
		for i = 0, 5, 0.1 do
			swait()
			local Alpha = .3
			RJ.C0 = clerp(RJ.C0,CFrame.new(2.74447132e-13, 0.00628674263, 4.19029675e-07, 0.99999994, 4.36557457e-11, 0, -4.3652193e-11, 0.999980211, -0.00628619269, 9.31322575e-10, 0.00628619175, 0.999980271),Alpha)
			LH.C0 = clerp(LH.C0,CFrame.new(-0.516345143, -0.986586034, 0.0229242463, 0.995020688, 0.0737848207, 0.0670047402, -0.0743697062, 0.997211039, 0.00627362682, -0.0663549677, -0.0112255123, 0.997732997),Alpha)
			RH.C0 = clerp(RH.C0,CFrame.new(0.477815509, -0.99112612, 0.0143765565, 0.99808538, -0.0329308398, -0.0523567572, 0.0333044007, 0.99942559, 0.00627828855, 0.0521199405, -0.0080099795, 0.998608768),Alpha)
			LS.C0 = clerp(LS.C0,CFrame.new(-1.42035723, 0.493732512, 0.0194591247, 0.987250268, 0.158408627, 0.0156119233, -0.158521742, 0.987335503, 0.00628757617, -0.0144182015, -0.00868224166, 0.999858439),Alpha)
			RS.C0 = clerp(RS.C0,CFrame.new(1.35784233, 0.380929202, -0.339150012, 0.890423656, 0.369434744, -0.265826464, -0.0434118584, -0.512461483, -0.857612193, -0.453057677, 0.775178194, -0.440269977),Alpha)
			NK.C0 = clerp(NK.C0,CFrame.new(5.94183803e-06, 1.49894607, -0.0144022629, 0.903856337, 0.0358069614, -0.426334888, 0.00745311938, 0.995022535, 0.0993709341, 0.427770972, -0.0929945856, 0.899090827),Alpha)
		end
		Sound(torso,429400881,1,1,false,true,true)
		for i = 0, 6, 0.1 do
			swait()
			local Alpha = .3
			RJ.C0 = clerp(RJ.C0,CFrame.new(2.74447132e-13, 0.00628674263, 4.19029675e-07, 0.99999994, 4.36557457e-11, 0, -4.3652193e-11, 0.999980211, -0.00628619269, 9.31322575e-10, 0.00628619175, 0.999980271),Alpha)
			LH.C0 = clerp(LH.C0,CFrame.new(-0.516345143, -0.986586034, 0.0229242463, 0.995020688, 0.0737848207, 0.0670047402, -0.0743697062, 0.997211039, 0.00627362682, -0.0663549677, -0.0112255123, 0.997732997),Alpha)
			RH.C0 = clerp(RH.C0,CFrame.new(0.477815509, -0.99112612, 0.0143765565, 0.99808538, -0.0329308398, -0.0523567572, 0.0333044007, 0.99942559, 0.00627828855, 0.0521199405, -0.0080099795, 0.998608768),Alpha)
			LS.C0 = clerp(LS.C0,CFrame.new(-1.42035723, 0.493732512, 0.0194591247, 0.987250268, 0.158408627, 0.0156119233, -0.158521742, 0.987335503, 0.00628757617, -0.0144182015, -0.00868224166, 0.999858439),Alpha)
			RS.C0 = clerp(RS.C0,CFrame.new(1.38204277, 0.275569797, -0.148523852, 0.434954822, 0.860323608, -0.265814841, 0.300874919, -0.417092472, -0.857617736, -0.848698258, 0.293047935, -0.440266252),Alpha)
			NK.C0 = clerp(NK.C0,CFrame.new(0.185457826, 1.49546897, -0.192251831, 0.865452588, 0.124405921, -0.485298753, 0.015648175, 0.961492956, 0.274383873, 0.500746369, -0.245060325, 0.83017987),Alpha)
		end
		hart:destroy()
		WalkSpeed = 16
		Hum.AutoRotate = true
		Hum.JumpPower = 50
		Attack = false
		NeutralAnims = true
	end
end

function YaYEET()
	Attack = true
	NeutralAnims = false
	WalkSpeed = 2
	for i = 0, 3, 0.1 do
		swait()
		local Alpha = .3
		RJ.C0 = clerp(RJ.C0,CFrame.new(-0.00286240783, 0.00628161477, -0.00104881125, 0.812644184, -0.00366364187, -0.582748652, 1.25324027e-06, 0.99998033, -0.00628495822, 0.582760096, 0.00510670478, 0.81262815),Alpha)
		LH.C0 = clerp(LH.C0,CFrame.new(-0.496485233, -0.990816236, 0.0216153599, 0.803447127, 0, 0.595376074, -0.00374295469, 0.99998033, 0.00505103637, -0.595364332, -0.00628670584, 0.803431273),Alpha)
		RH.C0 = clerp(RH.C0,CFrame.new(0.498535216, -0.990978837, 0.0154625224, 0.803447127, 0, 0.595376074, -0.00374295469, 0.99998033, 0.00505103637, -0.595364332, -0.00628670584, 0.803431273),Alpha)
		LS.C0 = clerp(LS.C0,CFrame.new(-1.54175317, 0.538998544, -0.350661546, 0.925994039, 0.377538294, -2.53279577e-05, -0.00177948072, 0.00429747393, -0.99998939, -0.377534091, 0.925984085, 0.0046512573),Alpha)
		RS.C0 = clerp(RS.C0,CFrame.new(1.2351619, 0.597906828, 0.279773176, 0.93086189, 0.365330189, -0.00547149777, 0.134925321, -0.329796135, 0.934360683, 0.339545637, -0.870499015, -0.356286913),Alpha)
		NK.C0 = clerp(NK.C0,CFrame.new(2.37277709e-06, 1.49894369, -0.0143988989, 0.803447127, 0, 0.595376074, -0.00374295469, 0.99998033, 0.00505103637, -0.595364332, -0.00628670584, 0.803431273),Alpha)
	end	
	repeat swait()
		local Alpha = .3
		RJ.C0 = clerp(RJ.C0,CFrame.new(-0.00286240783, 0.00628161477, -0.00104881125, 0.812644184, -0.00366364187, -0.582748652, 1.25324027e-06, 0.99998033, -0.00628495822, 0.582760096, 0.00510670478, 0.81262815),Alpha)
		LH.C0 = clerp(LH.C0,CFrame.new(-0.496485233, -0.990816236, 0.0216153599, 0.803447127, 0, 0.595376074, -0.00374295469, 0.99998033, 0.00505103637, -0.595364332, -0.00628670584, 0.803431273),Alpha)
		RH.C0 = clerp(RH.C0,CFrame.new(0.498535216, -0.990978837, 0.0154625224, 0.803447127, 0, 0.595376074, -0.00374295469, 0.99998033, 0.00505103637, -0.595364332, -0.00628670584, 0.803431273),Alpha)
		LS.C0 = clerp(LS.C0,CFrame.new(-1.54175317, 0.538998544, -0.350661546, 0.925994039, 0.377538294, -2.53279577e-05, -0.00177948072, 0.00429747393, -0.99998939, -0.377534091, 0.925984085, 0.0046512573),Alpha)
		RS.C0 = clerp(RS.C0,CFrame.new(1.2351619, 0.597906828, 0.279773176, 0.93086189, 0.365330189, -0.00547149777, 0.134925321, -0.329796135, 0.934360683, 0.339545637, -0.870499015, -0.356286913),Alpha)
		NK.C0 = clerp(NK.C0,CFrame.new(2.37277709e-06, 1.49894369, -0.0143988989, 0.803447127, 0, 0.595376074, -0.00374295469, 0.99998033, 0.00505103637, -0.595364332, -0.00628670584, 0.803431273),Alpha)
	until not S.UserInputService:IsMouseButtonPressed(Enum.UserInputType.MouseButton1)
	for i = 0, .7, 0.1 do
		swait()
		local Alpha = .3
		RJ.C0 = clerp(RJ.C0,CFrame.new(-0.248571783, 0.00628784206, -0.324586183, 0.811912656, 0.00367011107, 0.583767474, -1.25562076e-06, 0.99998033, -0.00628506951, -0.583779037, 0.00510219391, 0.811896563),Alpha)
		LH.C0 = clerp(LH.C0,CFrame.new(-0.496491075, -0.990815997, 0.0216309726, 0.82092762, 0, -0.571032405, 0.00358997518, 0.99998033, 0.00516101997, 0.57102108, -0.00628681574, 0.820911348),Alpha)
		RH.C0 = clerp(RH.C0,CFrame.new(0.498523057, -0.990978718, 0.0154775968, 0.82092762, 0, -0.571032405, 0.00358997518, 0.99998033, 0.00516101997, 0.57102108, -0.00628681574, 0.820911348),Alpha)
		LS.C0 = clerp(LS.C0,CFrame.new(-1.23150444, 0.50935328, 0.410490841, 0.925997376, -0.315732628, 0.206983045, -0.00177847152, 0.54460144, 0.838693202, -0.377526015, -0.776995718, 0.503737926),Alpha)
		RS.C0 = clerp(RS.C0,CFrame.new(0.944793224, 0.478973299, -0.436145425, 0.805186868, 0.59043932, 0.055278115, -0.0953396112, 0.220887601, -0.970628381, -0.5853073, 0.776266813, 0.234148055),Alpha)
		NK.C0 = clerp(NK.C0,CFrame.new(-6.4575579e-06, 1.49894357, -0.014398125, 0.82092762, 0, -0.571032405, 0.00358997518, 0.99998033, 0.00516101997, 0.57102108, -0.00628681574, 0.820911348),Alpha)
	end
	GrabbedHead.Parent = workspace
	GrabbedHead.CanCollide = true
	pcall(function() GrabbedHead.Weld:destroy() end)
	GrabbedHead.Velocity = Mouse.Hit.lookVector*250
	local ev;
	local haed = GrabbedHead;
	ev = GrabbedHead.Touched:connect(function(t)
		if(t.Parent and t.Parent ~= Char and not t.Parent:IsDescendantOf(Char) and t.Parent:FindFirstChildOfClass'Humanoid')then
			t.Parent:breakJoints()
		end
		if(ev and t.Parent ~= Char and not t.Parent:IsDescendantOf(Char))then 
			ev:disconnect() 
			ev = nil
			delay(2, function()
				for i = 0, 1, .05 do
					haed.Transparency = i
					swait()
				end
				haed:destroy()
			end)
		end
	end)
	GrabbedHead = nil
	for i = 0, 4, 0.1 do
		swait()
		local Alpha = .3
		RJ.C0 = clerp(RJ.C0,CFrame.new(-0.248571783, 0.00628784206, -0.324586183, 0.811912656, 0.00367011107, 0.583767474, -1.25562076e-06, 0.99998033, -0.00628506951, -0.583779037, 0.00510219391, 0.811896563),Alpha)
		LH.C0 = clerp(LH.C0,CFrame.new(-0.496491075, -0.990815997, 0.0216309726, 0.82092762, 0, -0.571032405, 0.00358997518, 0.99998033, 0.00516101997, 0.57102108, -0.00628681574, 0.820911348),Alpha)
		RH.C0 = clerp(RH.C0,CFrame.new(0.498523057, -0.990978718, 0.0154775968, 0.82092762, 0, -0.571032405, 0.00358997518, 0.99998033, 0.00516101997, 0.57102108, -0.00628681574, 0.820911348),Alpha)
		LS.C0 = clerp(LS.C0,CFrame.new(-1.23150444, 0.50935328, 0.410490841, 0.925997376, -0.315732628, 0.206983045, -0.00177847152, 0.54460144, 0.838693202, -0.377526015, -0.776995718, 0.503737926),Alpha)
		RS.C0 = clerp(RS.C0,CFrame.new(0.944793224, 0.478973299, -0.436145425, 0.805186868, 0.59043932, 0.055278115, -0.0953396112, 0.220887601, -0.970628381, -0.5853073, 0.776266813, 0.234148055),Alpha)
		NK.C0 = clerp(NK.C0,CFrame.new(-6.4575579e-06, 1.49894357, -0.014398125, 0.82092762, 0, -0.571032405, 0.00358997518, 0.99998033, 0.00516101997, 0.57102108, -0.00628681574, 0.820911348),Alpha)
	end
	Attack = false
	NeutralAnims = true
	WalkSpeed = 16
end


--// Wrap it all up \\--

Mouse.KeyDown:connect(function(k)
	if(Attack)then return end
	if(not GrabbedHead)then
		if(k == 'z')then
			ByeBye()
		elseif(k == 'x')then
			Decapitate()
		elseif(k == 'c')then
			TahHart()
		end
	end
end)

Mouse.Button1Down:connect(function()
	if(Attack)then return end
	if(GrabbedHead)then
		YaYEET()
	end	
end)

local deg = 0
while true do
	swait()
	Sine = Sine + Change
	if(not Music)then
		Music = Sound(Torso,MusicID,1,1,true,false,true)
		Music.Name = 'Music'
	end
	Music.SoundId = "rbxassetid://"..MusicID
	Music.Parent = Torso
	Music.Pitch = 1
	Music.Volume = 1
	if(not Muted)then
		Music:Resume()
	else
		Music:Pause()
	end
	
	
	if(God)then
		Hum.MaxHealth = 1e100
		Hum.Health = 1e100
		if(not Char:FindFirstChildOfClass'ForceField')then IN("ForceField",Char).Visible = false end
		Hum.Name = M.RNG()*100
	end
	
	local hitfloor,posfloor = workspace:FindPartOnRay(Ray.new(Root.CFrame.p,((CFrame.new(Root.Position,Root.Position - Vector3.new(0,1,0))).lookVector).unit * (4*PlayerSize)), Char)
	
	local Walking = (math.abs(Root.Velocity.x) > 1 or math.abs(Root.Velocity.z) > 1)
	local State = (Hum.PlatformStand and 'Paralyzed' or Hum.Sit and 'Sit' or not hitfloor and Root.Velocity.y < -1 and "Fall" or not hitfloor and Root.Velocity.y > 1 and "Jump" or hitfloor and Walking and (Hum.WalkSpeed < 24 and "Walk" or "Run") or hitfloor and "Idle")
	if(not Effects or not Effects.Parent)then
		Effects = IN("Model",Char)
		Effects.Name = "Effects"
	end																																																																																																				
	if(State == 'Run')then
		local wsVal = 7 / (Hum.WalkSpeed/16)
		local Alpha = math.min(.2 * (Hum.WalkSpeed/16),1)
		Change = 1
		RH.C1 = RH.C1:lerp(CF.N(0,1-.1*M.C(Sine/wsVal),0+.2*M.C(Sine/wsVal))*CF.A(M.R(8-0*M.C(Sine/wsVal))+-M.S(Sine/wsVal)/1.5,0,0),.2)
		LH.C1 = LH.C1:lerp(CF.N(0,1+.1*M.C(Sine/wsVal),0-.2*M.C(Sine/wsVal))*CF.A(M.R(8+0*M.C(Sine/wsVal))+M.S(Sine/wsVal)/1.5,0,0),.2)
	elseif(State == 'Walk')then
		local wsVal = 7 / (Hum.WalkSpeed/16)
		local Alpha = math.min(.2 * (Hum.WalkSpeed/16),1)
		Change = 1
		RH.C1 = RH.C1:lerp(CF.N(0,1-.5*M.C(Sine/wsVal)/2,0+.5*M.C(Sine/wsVal)/2)*CF.A(M.R(15-35*M.C(Sine/wsVal))+-M.S(Sine/wsVal)/2.5,0,0),Alpha)
		LH.C1 = LH.C1:lerp(CF.N(0,1+.5*M.C(Sine/wsVal)/2,0-.5*M.C(Sine/wsVal)/2)*CF.A(M.R(15+35*M.C(Sine/wsVal))+M.S(Sine/wsVal)/2.5,0,0),Alpha)
	else
		RH.C1 = RH.C1:lerp(CF.N(0,1,0),.2)
		LH.C1 = LH.C1:lerp(CF.N(0,1,0),.2)
	end
	Hum.WalkSpeed = WalkSpeed
	if(Remove_Hats)then Instance.ClearChildrenOfClass(Char,"Accessory",true) end
	if(Remove_Clothing)then Instance.ClearChildrenOfClass(Char,"Clothing",true) Instance.ClearChildrenOfClass(Char,"ShirtGraphic",true) end
	local face = Head:FindFirstChild'face'
	if(not face)then
		NewInstance("Decal",Head,{Name='face',Face=Enum.NormalId.Front,Texture="rbxassetid://404306534"})
	else
		face.Texture = "rbxassetid://404306534"
	end
	
	RArm.Color 		= C3.N(0.2,0.2,0.2)
	LArm.Color 		= C3.N(0.2,0.2,0.2)
	RLeg.Color 		= C3.N(0.2,0.2,0.2)
	LLeg.Color 		= C3.N(0.2,0.2,0.2)
	Torso.Color 	= C3.N(0.2,0.2,0.2)
	Head.Color 		= C3.N(0.2,0.2,0.2)
	
	deg = deg + 1
	HW.C0 = HW.C0:lerp(CF.N(0,1.5,0)*CF.A(0,M.R(deg),0),.2)
	if(NeutralAnims)then	
		if(State == 'Idle')then
			local Alpha = .1
			Change = 1
			RJ.C0 = RJ.C0:lerp(RJC0*CF.N(0,-.1+.05*M.C(Sine/18),0),Alpha)
			NK.C0 = NK.C0:lerp(NKC0*CF.A(M.R(-10-2.5*M.S(Sine/18)),M.R(20*M.C(Sine/18)),M.R(10)),Alpha)
			LS.C0 = LS.C0:lerp(LSC0*CF.N(0,0+.1*M.S(Sine/18),0)*CF.A(0,M.R(5+5*M.C(Sine/18)),M.R(-10-5*M.C(Sine/18))),Alpha)
			RS.C0 = RS.C0:lerp(RSC0*CF.N(0,0+.1*M.S(Sine/18),0)*CF.A(0,M.R(-5-5*M.C(Sine/18)),M.R(10+5*M.C(Sine/18))),Alpha)
			LH.C0 = LH.C0:lerp(LHC0*CF.N(0,0-.05*M.C(Sine/18),0)*CF.A(0,0,M.R(-10)),Alpha)
			RH.C0 = RH.C0:lerp(RHC0*CF.N(0,0-.05*M.C(Sine/18),-.2),Alpha)
			-- idle
		elseif(State == 'Run')then
			local wsVal = 7 / (Hum.WalkSpeed/16)
			local Alpha = math.min(.2 * (Hum.WalkSpeed/16),1)
			local Alpha2 = math.min(.15 * (Hum.WalkSpeed/16),1)
			RJ.C0 = RJ.C0:lerp(CF.N(0,0-.1*M.C(Sine/(wsVal/2)),0)*CF.A(M.R(-15+2.5*M.C(Sine/(wsVal/2))),M.R(8*M.C(Sine/wsVal)),0),Alpha2)
			NK.C0 = NK.C0:lerp(NKC0,Alpha)
			LS.C0 = LS.C0:lerp(LSC0*CF.N(0,0,0-.3*M.S(Sine/wsVal))*CF.A(M.R(0+45*M.S(Sine/wsVal)),0,M.R(-5)),Alpha)
			RS.C0 = RS.C0:lerp(RSC0*CF.N(0,0,0+.3*M.S(Sine/wsVal))*CF.A(M.R(0-45*M.S(Sine/wsVal)),0,M.R(5)),Alpha)
			LH.C0 = LH.C0:lerp(LHC0*CF.N(0,0+.1*M.C(Sine/(wsVal/2)),0)*CF.A(0,-M.R(4*M.C(Sine/wsVal)),0),Alpha)
			RH.C0 = RH.C0:lerp(RHC0*CF.N(0,0+.1*M.C(Sine/(wsVal/2)),0)*CF.A(0,-M.R(4*M.C(Sine/wsVal)),0),Alpha)
		elseif(State == 'Walk')then
			local wsVal = 7 / (Hum.WalkSpeed/16)
			local Alpha = math.min(.2 * (Hum.WalkSpeed/16),1)
			local Alpha2 = math.min(.15 * (Hum.WalkSpeed/16),1)
			RJ.C0 = RJ.C0:lerp(CF.N(0,-.175+.1*M.C(Sine/(wsVal/2)+-M.S(Sine/(wsVal/2))/7),0)*CF.A(M.R(-9-2.5*M.C(Sine/(wsVal/2))),M.R(10*M.C(Sine/wsVal)),Root.RotVelocity.y/75),Alpha2)
			NK.C0 = NK.C0:lerp(NKC0*CF.A(0,-Head.RotVelocity.y/75,0),Alpha)
			LS.C0 = LS.C0:lerp(LSC0*CF.N(0,0,-.27*M.C(Sine/wsVal))*CF.A(M.R(45*M.C(Sine/wsVal)),0,M.R(-5)+LArm.RotVelocity.y/75),Alpha)
			RS.C0 = RS.C0:lerp(RSC0*CF.N(0,0,.27*M.C(Sine/wsVal))*CF.A(M.R(-45*M.C(Sine/wsVal)),0,M.R(5)-RArm.RotVelocity.y/75),Alpha)
			LH.C0 = LH.C0:lerp(LHC0*CF.N(0,0-.1*M.C(Sine/(wsVal/2)),0)*CF.A(0,0,0),Alpha)
			RH.C0 = RH.C0:lerp(RHC0*CF.N(0,0-.1*M.C(Sine/(wsVal/2)),0)*CF.A(0,0,0),Alpha)
		elseif(State == 'Jump')then
			local Alpha = .1
			local idk = math.min(math.max(Root.Velocity.Y/50,-M.R(90)),M.R(90))
			LS.C0 = LS.C0:lerp(LSC0*CF.A(M.R(-5),0,M.R(-90)),Alpha)
			RS.C0 = RS.C0:lerp(RSC0*CF.A(M.R(-5),0,M.R(90)),Alpha)
			RJ.C0 = RJ.C0:lerp(RJC0*CF.A(math.min(math.max(Root.Velocity.Y/100,-M.R(45)),M.R(45)),0,0),Alpha)
			NK.C0 = NK.C0:lerp(NKC0*CF.A(math.min(math.max(Root.Velocity.Y/100,-M.R(45)),M.R(45)),0,0),Alpha)
			LH.C0 = LH.C0:lerp(LHC0*CF.A(0,0,M.R(-5)),Alpha)
			RH.C0 = RH.C0:lerp(RHC0*CF.N(0,1,-1)*CF.A(M.R(-5),0,M.R(5)),Alpha)
		elseif(State == 'Fall')then
			local Alpha = .1
			local idk = math.min(math.max(Root.Velocity.Y/50,-M.R(90)),M.R(90))
			LS.C0 = LS.C0:lerp(LSC0*CF.A(M.R(-5),0,M.R(-90)+idk),Alpha)
			RS.C0 = RS.C0:lerp(RSC0*CF.A(M.R(-5),0,M.R(90)-idk),Alpha)
			RJ.C0 = RJ.C0:lerp(RJC0*CF.A(math.min(math.max(Root.Velocity.Y/100,-M.R(45)),M.R(45)),0,0),Alpha)
			NK.C0 = NK.C0:lerp(NKC0*CF.A(math.min(math.max(Root.Velocity.Y/100,-M.R(45)),M.R(45)),0,0),Alpha)
			LH.C0 = LH.C0:lerp(LHC0*CF.A(0,0,M.R(-5)),Alpha)
			RH.C0 = RH.C0:lerp(RHC0*CF.N(0,1,-1)*CF.A(M.R(-5),0,M.R(5)),Alpha)
		elseif(State == 'Paralyzed')then
			-- paralyzed
		elseif(State == 'Sit')then
			-- sit
		end
	end
	
end

