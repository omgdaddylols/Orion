local OrionLib = loadstring(game:HttpGet(('https://pastebin.com/raw/ua6CXjPi')))()
local Window = OrionLib:MakeWindow({Name = "TechnicalHub", HidePremium = false, SaveConfig = true, ConfigFolder = "OrionTest"})

--[[
Name = <string> - The name of the UI.
HidePremium = <bool> - Whether or not the user details shows Premium status or not.
SaveConfig = <bool> - Toggles the config saving in the UI.
ConfigFolder = <string> - The name of the folder where the configs are saved.
IntroEnabled = <bool> - Whether or not to show the intro animation.
IntroText = <string> - Text to show in the intro animation.
IntroIcon = <string> - URL to the image you want to use in the intro animation.
Icon = <string> - URL to the image you want displayed on the window.
CloseCallback = <function> - Function to execute when the window is closed.
]]
local Tab = Window:MakeTab({
	Name = "Main",
	Icon = "rbxassetid://4483345998",
	PremiumOnly = false
})
local Section = Tab:AddSection({
	Name = "Aim and ESP"
})

--[[
Name = <string> - The name of the section.
]]
OrionLib:MakeNotification({
	Name = "Config Loaded!",
	Content = "Notification content... Welcome To TechnicalHub!",
	Image = "rbxassetid://4483345998",
	Time = 5
})

--[[
Title = <string> - The title of the notification.
Content = <string> - The content of the notification.
Image = <string> - The icon of the notification.
Time = <number> - The duration of the notfication.
]]

Tab:AddButton({
	Name = "Aimbot!",
	Callback = function()
      		--[[
    AimHot v8, Tech#1182

    
]]



-- Extremly bad code starts below here

local DEBUG_MODE = false -- warnings, prints and profiles dont change idiot thanks

-- Ok I declare some variables here for micro optimization. I might declare again in the blocks because I am lazy to check here
local game, workspace = game, workspace

local cf, v3, v2, udim2 = CFrame, Vector3, Vector2, UDim2
local string, math, table, Color3, tonumber, tostring = string, math, table, Color3, tonumber, tostring

local cfnew = cf.new
local cf0 = cfnew()

local v3new = v3.new
local v30 = v3new()

local v2new = v2.new
local v20 = v2new()

local setmetatable = setmetatable
local getmetatable = getmetatable

local type, typeof = type, typeof

local Instance = Instance

local drawing = Drawing or drawing

local mousemoverel = mousemoverel or (Input and Input.MouseMove)

local readfile = readfile
local writefile = writefile
local appendfile = appendfile

local warn, print = DEBUG_MODE and warn or function() end, DEBUG_MODE and print or function() end


local required = {
    mousemoverel, drawing, readfile, writefile, appendfile, game.HttpGet, game.GetObjects
}

for i,v in pairs(required) do
    if v == nil then
        warn("Your exploit is not supported (may consider purchasing a better one?)!")
        return -- Only pros return in top-level function
    end
end

local servs
servs = setmetatable(
{
    Get = function(self, serv)
        if servs[serv] then return servs[serv] end
        local s = game:GetService(serv)
        if s then servs[serv] = s end
        return s
    end;
}, {
    __index = function(self, index)
        local s = game:GetService(index)
        if s then servs[index] = s end
        return s
    end;
})

local connections = {}
local function bindEvent(event, callback) -- Let me disconnect in peace
    local con = event:Connect(callback)
    table.insert(connections, con)
    return con
end

local players = servs.Players
local runservice = servs.RunService
local http = servs.HttpService
local uis = servs.UserInputService

local function jsonEncode(t)
    return http:JSONEncode(t)
end
local function jsonDecode(t)
    return http:JSONDecode(t)
end

local function existsFile(name)
    return pcall(function()
        return readfile(name)
    end)
end

local function mergetab(a,b)
    local c = a or {}
    for i,v in pairs(b or {}) do 
        c[i] = v 
    end
    return c
end

local locpl = players.LocalPlayer
local mouse = locpl:GetMouse()
local camera = workspace.CurrentCamera
workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(function() -- if a script changes currentcamera
    camera = workspace.CurrentCamera
end)


local findFirstChild = game.FindFirstChild
local findFirstChildOfClass = game.FindFirstChildOfClass
local isDescendantOf = game.IsDescendantOf

-- Just to check another aimhot instance is running and close it
local uid = tick() .. math.random(1,100000) .. math.random(1,100000)
if shared.ah8 and shared.ah8.close and shared.ah8.uid~=uid then shared.ah8:close() end

-- Main shitty script should start below here

warn("AH8_MAIN : Running script...")

local event = {} 
local utility = {}
local serializer = {}

local settings = {}

local hud = loadstring(game:HttpGet("https://pastebin.com/raw/3hREvLEU", DEBUG_MODE == false and true or DEBUG_MODE == true and false))()[1] -- Ugly ui do not care

local aimbot = {}

local visuals = {}

local crosshair = {}
local esp = {}
local boxes = {}
local tracers = {}

local run = {}
local ah8 = {enabled = true;}


local visiblekids = {} -- no need to check twice each frame yes? todo :(
-- Some libraries

do
    --/ Events : custom event system, bindables = gay

    local type = type;
    local coroutine = coroutine;
    local create = coroutine.create;
    local resume = coroutine.resume;

    local function spawn(f, ...)
        resume(create(f), ...)
    end

    function event.new(t)
        local self = t or {}
        
        local n = 0
        local connections = {}
        function self:connect(func)
            if type(func) ~= "function" then return end

            n = n + 1
            local my = n
            connections[n] = func

            local connected = true
            local function disconnect()
                if connected ~= true then return end
                connected = false

                connections[n] = nil
            end

            return disconnect
        end


        local function fire(...)
            for i,v in pairs(connections) do
                v(...)
            end
        end

        return fire, self
    end
end

do
    --/ Utility : To make it easier for me to edit

    local getPlayers = players.GetPlayers
    local getPartsObscuringTarget = camera.GetPartsObscuringTarget
    local worldToViewportPoint = camera.WorldToViewportPoint
    local worldToScreenPoint = camera.WorldToScreenPoint
    local raynew = Ray.new
    local findPartOnRayWithIgnoreList = workspace.FindPartOnRayWithIgnoreList
    local findPartOnRay = workspace.FindPartOnRay
    local findFirstChild = game.FindFirstChild

    local function raycast(ray, ignore, callback)
        local ignore = ignore or {}

        local hit, pos, normal, material = findPartOnRayWithIgnoreList(workspace, ray, ignore)
        while hit and callback do
            local Continue, _ignore = callback(hit)
            if not Continue then
                break
            end
            if _ignore then
                table.insert(ignore, _ignore)
            else
                table.insert(ignore, hit)
            end
            hit, pos, normal, material = findPartOnRayWithIgnoreList(workspace, ray, ignore)
        end
        return hit, pos, normal, material
    end

    local function badraycastnotevensure(pos, ignore) -- 1 ray > 1 obscuringthing | 100 rays < 1 obscuring thing
        local hitparts = getPartsObscuringTarget(camera, {pos}, ignore or {})
        return hitparts
    end

    local charshit = {}
    function utility.getcharacter(player) -- Change this or something if you want to add support for other games.
        if (player == nil) then return end
        if (charshit[player]) then return charshit[player] end

        local char = player.Character
        if (char == nil or isDescendantOf(char, game) == false) then
            char = findFirstChild(workspace, player.Name)
        end

        return char
    end

    utility.mychar = nil
    utility.myroot = nil

    local rootshit = {}
    function utility.getroot(player)
        if (player == nil) then return end
        if (rootshit[player]) then return rootshit[player] end

        local char
        if (player:IsA("Player")) then
            char = utility.getcharacter(player)
        else
            char = player
        end

        if (char ~= nil) then
            local root = (findFirstChild(char, "HumanoidRootPart") or char.PrimaryPart)
            if (root ~= nil) then -- idk
                --bindEvent(root.AncestryChanged, function(_, parent)
                --    if (parent == nil) then
                --        roostshit[player] = nil
                --    end
                --end)
            end

            --rootshit[player] = root
            return root
        end

        return
    end

    spawn(function()
        while ah8 and ah8.enabled do -- Some games are gay
            utility.mychar = utility.getcharacter(locpl)
            if (utility.mychar ~= nil) then
                utility.myroot = utility.getroot(locpl)
            end
            wait(.5)
        end
    end)
    utility.mychar = locpl.Character
    utility.myroot = utility.mychar and findFirstChild(utility.mychar, "HumanoidRootPart") or utility.mychar and utility.mychar.PrimaryPart
    bindEvent(locpl.CharacterAdded, function(char)
        utility.mychar = char
        wait(.1)
        utility.myroot = utility.mychar and findFirstChild(utility.mychar, "HumanoidRootPart") or utility.mychar.PrimaryPart
    end)
    bindEvent(locpl.CharacterRemoving, function()
        utility.mychar = nil
        utility.myroot = nil
    end)
    

    function utility.isalive(_1, _2)
        if _1 == nil then return end
        local Char, RootPart
        if _2 ~= nil then
            Char, RootPart = _1,_2
        else
            Char = utility.getcharacter(_1)
            RootPart = Char and (Char:FindFirstChild("HumanoidRootPart") or Char.PrimaryPart)
        end

        if Char and RootPart then
            local Human = findFirstChildOfClass(Char, "Humanoid")
            if RootPart and Human then
                if Human.Health > 0 then
                    return true
                end
            elseif RootPart and isDescendantOf(Char, game) then
                return true
            end
        end

        return false
    end

    local shit = false
    function utility.isvisible(char, root, max, ...)
        local pos = root.Position
        if shit or max > 4 then
            local parts = badraycastnotevensure(pos, {utility.mychar, ..., camera, char, root})
            
            return parts == 0
        else
            local camp = camera.CFrame.p
            local dist = (camp - pos).Magnitude

            local hitt = 0
            local hit = raycast(raynew(camp, (pos - camp).unit * dist), {utility.mychar, ..., camera}, function(hit)

                if hit.CanCollide ~= false then-- hit.Transparency ~= 1 then¨
                    hitt = hitt + 1
                    return hitt < max
                end
            
                if isDescendantOf(hit, char) then
                    return
                end
                return true
            end)

            return hit == nil and true or isDescendantOf(hit, char), hitt
        end
    end
    function utility.sameteam(player, p1)
        local p0 = p1 or locpl
        return (player.Team~=nil and player.Team==p0.Team) and player.Neutral == false or false
    end
    function utility.getDistanceFromMouse(position)
        local screenpos, vis = worldToViewportPoint(camera, position)
        if vis and screenpos.Z > 0 then
            return (v2new(mouse.X, mouse.Y) - v2new(screenpos.X, screenpos.Y)).Magnitude
        end
        return math.huge
    end


    local hashes = {}
    function utility.getClosestMouseTarget(settings)
        local closest, temp = nil, settings.fov or math.huge
        local plr

        for i,v in pairs(getPlayers(players)) do
            if (locpl ~= v and (settings.ignoreteam==true and utility.sameteam(v)==false or settings.ignoreteam == false)) then
                local character = utility.getcharacter(v)
                if character and isDescendantOf(character, game) == true then
                    local hash = hashes[v]
                    local part = hash or findFirstChild(character, settings.name or "HumanoidRootPart") or findFirstChild(character, "HumanoidRootPart") or character.PrimaryPart
                    if hash == nil then hashes[v] = part end
                    if part and isDescendantOf(part, game) == true then
                        local legal = true

                        local rp = part:GetRenderCFrame().p
                        local distance = utility.getDistanceFromMouse(rp)
                        if temp <= distance then
                            legal = false
                        end

                        if legal then
                            if settings.checkifalive then
                                local isalive = utility.isalive(character, part)
                                if not isalive then
                                    legal = false
                                end
                            end
                        end

                        if legal then
                            local visible = true
                            if settings.ignorewalls == false then
                                local vis = utility.isvisible(character, part, (settings.maxobscuringparts or 0))
                                if not vis then
                                    legal = false
                                end
                            end
                        end

                        if legal then
                            local dist1
                            temp = distance
                            closest = part
                            plr = v
                        end
                    end
                end
            end
        end -- who doesnt love 5 ends in a row?

        return closest, temp, plr
    end
    function utility.getClosestTarget(settings)

        local closest, temp = nil, math.huge
        --local utility.myroot = utility.mychar and (findFirstChild(utility.mychar, settings.name or "HumanoidRootPart") or findFirstChild(utility.mychar, "HumanoidRootPart"))
        
        if utility.myroot then
            for i,v in pairs(getPlayers(players)) do
                if (locpl ~= v) and (settings.ignoreteam==true and utility.sameteam(v)==false or settings.ignoreteam == false) then
                    local character = utility.getcharacter(v)
                    if character then
                        local hash = hashes[v]
                        local part = hash or findFirstChild(character, settings.name or "HumanoidRootPart") or findFirstChild(character, "HumanoidRootPart")
                        if hash == nil then hashes[v] = part end

                        if part then
                            local visible = true
                            if settings.ignorewalls == false then
                                local vis, p = utility.isvisible(character, part, (settings.maxobscuringparts or 0))
                                if p <= (settings.maxobscuringparts or 0) then
                                    visible = vis
                                end
                            end

                            if visible then
                                local distance = (utility.myroot.Position - part.Position).Magnitude
                                if temp > distance then
                                    temp = distance
                                    closest = part
                                end
                            end
                        end
                    end
                end
            end
        end

        return closest, temp
    end

    spawn(function()
        while ah8 and ah8.enabled do
            for i,v in pairs(hashes) do
                hashes[i] = nil
                wait()
            end
            wait(4)
            --hashes = {}
        end
    end)
end


local serialize
local deserialize
do
    --/ Serializer : garbage : slow as fuck
	
	local function hex_encode(IN, len)
	    local B,K,OUT,I,D=16,"0123456789ABCDEF","",0,nil
	    while IN>0 do
	        I=I+1
	        IN,D=math.floor(IN/B), IN%B+1
	        OUT=string.sub(K,D,D)..OUT
	    end
		if len then
			OUT = ('0'):rep(len - #OUT) .. OUT
		end
	    return OUT
	end
	local function hex_decode(IN) 
		return tonumber(IN, 16) 
	end

    local types = {
        ["nil"] = "0";
        ["boolean"] = "1";
        ["number"] = "2";
        ["string"] = "3";
        ["table"] = "4";

		["Vector3"] = "5";
		["CFrame"] = "6";
        ["Instance"] = "7";
	
		["Color3"] = "8";
    }
    local rtypes = (function()
        local a = {}
        for i,v in pairs(types) do
            a[v] = i
        end
        return a
    end)()

    local typeof = typeof or type
    local function encode(t, ...)
        local type = typeof(t)
        local s = types[type]
        local c = ''
        if type == "nil" then
            c = types[type] .. "0"
        elseif type == "boolean" then
            local t = t == true and '1' or '0'
            c = s .. t
        elseif type == "number" then
            local new = tostring(t)
            local len = #new
            c = s .. len .. "." .. new
        elseif type == "string" then
            local new = t
            local len = #new
            c = s .. len .. "." .. new
		elseif type == "Vector3" then
			local x,y,z = tostring(t.X), tostring(t.Y), tostring(t.Z)
			local new = hex_encode(#x, 2) .. x .. hex_encode(#y, 2) .. y .. hex_encode(#z, 2) .. z
			c = s .. new
		elseif type == "CFrame" then
			local a = {t:GetComponents()}
			local new = ''
			for i,v in pairs(a) do
				local l = tostring(v)
				new = new .. hex_encode(#l, 2) .. l
			end
			c = s .. new
		elseif type == "Color3" then
			local a = {t.R, t.G, t.B}
			local new = ''
			for i,v in pairs(a) do
				local l = tostring(v)
				new = new .. hex_encode(#l, 2) .. l
			end
			c = s .. new
        elseif type == "table" then
            return serialize(t, ...)
        end
        return c
    end
    local function decode(t, extra)
        local p = 0
        local function read(l)
            l = l or 1
            p = p + l
            return t:sub(p-l + 1, p)
        end
        local function get(a)
            local k = ""
            while p < #t do
                if t:sub(p+1,p+1) == a then
                    break
                else
                    k = k .. read()
                end
            end
            return k
        end
        local type = rtypes[read()]
        local c

        if type == "nil" then
            read()
        elseif type == "boolean" then
            local d = read()
            c = d == "1" and true or false
        elseif type == "number" then
            local length = tonumber(get("."))
            local d = read(length+1):sub(2,-1)
            c = tonumber(d)
        elseif type == "string" then
            local length = tonumber(get(".")) --read()
            local d = read(length+1):sub(2,-1)
            c = d
		elseif type == "Vector3" then
			local function getnext()
				local length = hex_decode(read(2))
				local a = read(tonumber(length))
				return tonumber(a)
			end
			local x,y,z = getnext(),getnext(),getnext()
			c = Vector3.new(x, y, z)
		elseif type == "CFrame" then
			local a = {}
			for i = 1,12 do
				local l = hex_decode(read(2))
				local b = read(tonumber(l))
				a[i] = tonumber(b)
			end
			c = CFrame.new(unpack(a))
        elseif type == "Instance" then
			local pos = hex_decode(read(2))
			c = extra[tonumber(pos)]
		elseif type == "Color3" then
			local a = {}
			for i = 1,3 do
				local l = hex_decode(read(2))
				local b = read(tonumber(l))
				a[i] = tonumber(b)
			end
			c = Color3.new(unpack(a))
        end
        return c
    end

    function serialize(data, p)
		if data == nil then return end
        local type = typeof(data)
        if type == "table" then
            local extra = {}
            local s = types[type]
            local new = ""
            local p = p or 0
            for i,v in pairs(data) do
                local i1,v1
                local t0,t1 = typeof(i), typeof(v)

				local a,b
                if t0 == "Instance" then
                    p = p + 1
                    extra[p] = i
                    i1 = types[t0] .. hex_encode(p, 2)
                else
                    i1, a = encode(i, p)
					if a then
						for i,v in pairs(a) do
							extra[i] = v
						end
					end
                end
                
                if t1 == "Instance" then
                    p = p + 1
                    extra[p] = v
                    v1 = types[t1] .. hex_encode(p, 2)
                else
                    v1, b = encode(v, p)
					if b then
						for i,v in pairs(b) do
							extra[i] = v
						end
					end
                end
                new = new .. i1 .. v1
            end
            return s .. #new .. "." .. new, extra
		elseif type == "Instance" then
			return types[type] .. hex_encode(1, 2), {data}
        else
            return encode(data), {}
        end
    end

    function deserialize(data, extra)
		if data == nil then return end
		extra = extra or {}
		
        local type = rtypes[data:sub(1,1)]
        if type == "table" then

            local p = 0
            local function read(l)
                l = l or 1
                p = p + l
                return data:sub(p-l + 1, p)
            end
            local function get(a)
                local k = ""
                while p < #data do
                    if data:sub(p+1,p+1) == a then
                        break
                    else
                        k = k .. read()
                    end
                end
                return k
            end

            local length = tonumber(get("."):sub(2, -1))
            read()

            local new = {}

            local l = 0
            while p <= length do
                l = l + 1

				local function getnext()
					local i
                    local t = read()
                    local type = rtypes[t]

                    if type == "nil" then
                        i = decode(t .. read())
                    elseif type == "boolean" then
                        i = decode(t .. read())
                    elseif type == "number" then
                        local l = get(".")
                        
                        local dc = t .. l .. read()
                        local a = read(tonumber(l))
                        dc = dc .. a

                        i = decode(dc)
                 	elseif type == "string" then
                        local l = get(".")
                        local dc = t .. l .. read()
                        local a = read(tonumber(l))
                        dc = dc .. a

                        i = decode(dc)
					 elseif type == "Vector3" then
						local function getnext()
							local length = hex_decode(read(2))
							local a = read(tonumber(length))
							return tonumber(a)
						end
						local x,y,z = getnext(),getnext(),getnext()
						i = Vector3.new(x, y, z)
					elseif type == "CFrame" then
						local a = {}
						for i = 1,12 do
							local l = hex_decode(read(2))
							local b = read(tonumber(l)) -- why did I decide to do this
							a[i] = tonumber(b)
						end
						i = CFrame.new(unpack(a))
					elseif type == "Instance" then
						local pos = hex_decode(read(2))
						i = extra[tonumber(pos)]
					elseif type == "Color3" then
						local a = {}
						for i = 1,3 do
							local l = hex_decode(read(2))
							local b = read(tonumber(l))
							a[i] = tonumber(b)
						end
						i = Color3.new(unpack(a))
                    elseif type == "table" then
                        local l = get(".")
                        local dc = t .. l .. read() .. read(tonumber(l))
                        i = deserialize(dc, extra)
                    end
					return i
				end
                local i = getnext()
                local v = getnext()

               new[(typeof(i) ~= "nil" and i or l)] =  v
            end


            return new
		elseif type == "Instance" then
			local pos = tonumber(hex_decode(data:sub(2,3)))
			return extra[pos]
        else
            return decode(data, extra)
        end
    end
end


-- great you have come a far way now stop before my horrible scripting will infect you moron

do
    --/ Settings

    -- TODO: Other datatypes.
    settings.fileName = "AimBot_v1_settings.txt" -- Lovely
    settings.saved = {}

    function settings:Get(name, default)
        local self = {}
        local value = settings.saved[name]
        if value == nil and default ~= nil then
            value = default
            settings.saved[name] = value
        end
        self.Value = value
        function self:Set(val)
            self.Value = val
            settings.saved[name] = val
        end
        return self  --value or default
    end

    function settings:Set(name, value)
        local r = settings.saved[name]
        settings.saved[name] = value
        return r
    end

    function settings:Save()
        local savesettings = settings:GetAll() or {}
        local new = mergetab(savesettings, settings.saved)
        local js = serialize(new)

        writefile(settings.fileName, js)
    end

    function settings:GetAll()
        if not existsFile(settings.fileName) then
            return
        end
        local fileContents = readfile(settings.fileName)

        local data
        pcall(function()
            data = deserialize(fileContents)
        end)
        return data
    end

    function settings:Load()
        if not existsFile(settings.fileName) then
            return
        end
        local fileContents = readfile(settings.fileName)

        local data
        pcall(function()
            data = deserialize(fileContents)
        end)

        if data then
            data = mergetab(settings.saved, data)
        end
        settings.saved = data
        return data
    end
    settings:Load()

    spawn(function()
        while ah8 and ah8.enabled do
            settings:Save()
            wait(5)
        end
    end)
end

-- Aiming aim bot aim aim stuff bot

do
    --/ Aimbot

    -- Do I want to make this decent?
    local aimbot_settings = {}
    aimbot_settings.ignoreteam = settings:Get("aimbot.ignoreteam", false)
    aimbot_settings.sensitivity = settings:Get("aimbot.sensitivity", .5)
    aimbot_settings.locktotarget = settings:Get("aimbot.locktotarget", true)
    aimbot_settings.checkifalive = settings:Get("aimbot.checkifalive", false)

    aimbot_settings.ignorewalls = settings:Get("aimbot.ignorewalls", true)
    aimbot_settings.maxobscuringparts = settings:Get("aimbot.maxobscuringparts", 0)


    aimbot_settings.enabled = settings:Get("aimbot.enabled", false)
    aimbot_settings.keybind = settings:Get("aimbot.keybind", "MouseButton2")
    aimbot_settings.presstoenable = settings:Get("aimbot.presstoenable", true)

    aimbot_settings.fovsize = settings:Get("aimbot.fovsize", 400)
    aimbot_settings.fovenabled = settings:Get("aimbot.fovenabled", true)
    aimbot_settings.fovsides = settings:Get("aimbot.fovsides", 10)
    aimbot_settings.fovthickness = settings:Get("aimbot.fovthickness", 1)
    
    aimbot.fovshow = aimbot_settings.fovenabled.Value

    setmetatable(aimbot, {
        __index = function(self, index)
            if aimbot_settings[index] ~= nil then
                local Value = aimbot_settings[index]
                if typeof(Value) == "table" then
                    return typeof(Value) == "table" and Value.Value
                else
                    return Value
                end
            end
            warn(("AH8_ERROR : AimbotSettings : Tried to index %s"):format(tostring(index)))
        end;
        __newindex = function(self, index, value)
            if typeof(value) ~= "function" then
                if aimbot_settings[index] then
                    local v = aimbot_settings[index]
                    if typeof(v) ~= "table" then
                        aimbot_settings[index] = value
                        return
                    elseif v.Set then
                        v:Set(value)
                        return
                    end
                end
            end
            rawset(self, index, value)
        end; -- ew
    })


    local worldToScreenPoint = camera.WorldToScreenPoint -- why did I start this
    local target, _, closestplr = nil, nil, nil;
    local completeStop = false

    local enabled = false
    bindEvent(uis.InputBegan, function(key,gpe)
        if aimbot.enabled == false then return end

        if aimbot.presstoenable then
            aimbot.fovshow = true
        else
            aimbot.fovshow = enabled == true
        end

        local keyc = key.KeyCode == Enum.KeyCode.Unknown and key.UserInputType or key.KeyCode
        if keyc.Name == aimbot.keybind then
            if aimbot.presstoenable then
                enabled = true
                aimbot.fovshow = true
            else
                enabled = not enabled
                aimbot.fovshow = enabled == true
            end
        end
    end)
    bindEvent(uis.InputEnded, function(key)
        if aimbot.enabled == false then enabled = false aimbot.fovshow = false end
        if aimbot.presstoenable then
            aimbot.fovshow = true
        else
            aimbot.fovshow = enabled == true
        end

        local keyc = key.KeyCode == Enum.KeyCode.Unknown and key.UserInputType or key.KeyCode
        if keyc.Name == aimbot.keybind then
            if aimbot.presstoenable then
                enabled = false
            end
        end
    end)


    local function calculateTrajectory()
        -- my math is a bit rusty atm
    end

    local function aimAt(vector)
        if completeStop then return end
        local newpos = worldToScreenPoint(camera, vector)
        mousemoverel((newpos.X - mouse.X) * aimbot.sensitivity, (newpos.Y - mouse.Y) * aimbot.sensitivity)
    end

    function aimbot.step()
        if completeStop or aimbot.enabled == false or enabled == false or utility.mychar == nil or isDescendantOf(utility.mychar, game) == false then 
            if target or closestplr then
                target, closestplr, _ = nil, nil, _
            end
            return 
        end
        
        if aimbot.locktotarget == true then
            local cchar = utility.getcharacter(closestplr)
            if target == nil or isDescendantOf(target, game) == false or closestplr == nil or closestplr.Parent == nil or cchar  == nil or isDescendantOf(cchar, game) == false then
                target, _, closestplr = utility.getClosestMouseTarget({ -- closest to mouse or camera mode later just wait
                    ignoreteam = aimbot.ignoreteam;
                    ignorewalls = aimbot.ignorewalls;
                    maxobscuringparts = aimbot.maxobscuringparts;
                    name = 'Head';
                    fov = aimbot.fovsize;
                    checkifalive = aimbot.checkifalive;
                    -- mode = "mouse";
                })
            else
                --target = target
                local stop = false
                if stop == false and not (aimbot.ignoreteam==true and utility.sameteam(closestplr)==false or aimbot.ignoreteam == false) then
                    stop = true
                end
                local visible = true

                if stop == false and aimbot.ignorewalls == false then
                    local vis = utility.isvisible(target.Parent, target, (aimbot.maxobscuringparts or 0))
                    if not vis then
                        stop = true
                    end
                end

                if stop == false and aimbot.checkifalive then
                    local isalive = utility.isalive(character, part)
                    if not isalive then
                        stop = true
                    end
                end

                if stop then
                    -- getClosestTarget({mode = "mouse"}) later
                    target, _, closestplr = utility.getClosestMouseTarget({
                        ignoreteam = aimbot.ignoreteam;
                        ignorewalls = aimbot.ignorewalls;
                        maxobscuringparts = aimbot.maxobscuringparts;
                        name = 'Head';
                        fov = aimbot.fovsize;
                        checkifalive = aimbot.checkifalive;
                    })
                end
            end
        else
            target = utility.getClosestMouseTarget({
                ignoreteam = aimbot.ignoreteam;
                ignorewalls = aimbot.ignorewalls;
                maxobscuringparts = aimbot.maxobscuringparts;
                name = 'Head';
                fov = aimbot.fovsize;
                checkifalive = aimbot.checkifalive;
            })
        end

        if target then
            aimAt(target:GetRenderCFrame().Position)
            -- hot or not?
        end
    end

    function aimbot:End()
        completeStop = true
        target = nil
    end
end


-- Mostly visuals below here
local clearDrawn, newdrawing
do
    --/ Drawing extra functions

    local insert = table.insert
    local newd = drawing.new

    local drawn = {}
    function clearDrawn() -- who doesnt love drawing library
        for i,v in pairs(drawn) do
            pcall(function() v:Remove() end)
            drawn[i] = nil
        end
        drawn = {}
    end

    function newdrawing(class, props)
        --if visuals.enabled ~= true then
        --    return
        --end
        local new = newd(class)
        for i,v in pairs(props) do
            new[i] = v
        end
        insert(drawn, new)
        return new
    end
end


do
    --/ Crosshair
    local crosshair_settings = {}
    crosshair_settings.enabled = settings:Get("crosshair.enabled", false)
    crosshair_settings.size = settings:Get("crosshair.size", 40)
    crosshair_settings.thickness = settings:Get("crosshair.thickness", 1)
    crosshair_settings.color = Color3.fromRGB(255,0,0)
    crosshair_settings.transparency = settings:Get("crosshair.transparency", .1)

    setmetatable(crosshair, { -- yes I know it is easier ways to add this but that requires effort
        __index = function(self, index)
            if crosshair_settings[index] ~= nil then
                local Value = crosshair_settings[index]
                if typeof(Value) == "table" then
                    return typeof(Value) == "table" and Value.Value
                else
                    return Value
                end
            end
            warn(("AH8_ERROR : CrosshairSettings : Tried to index %s"):format(tostring(index)))
        end;
        __newindex = function(self, index, value)
            if typeof(value) ~= "function" then
                if crosshair_settings[index] then
                    local v = crosshair_settings[index]
                    if typeof(v) ~= "table" then
                        crosshair_settings[index] = value
                        return
                    elseif v.Set then
                        v:Set(value)
                        return
                    end
                end
            end
            rawset(self, index, value)
        end;
    })

    local crossHor
    local crossVer
    local vpSize = camera.ViewportSize



    local completeStop = false
    local function drawCrosshair()
        if completeStop then return crosshair:Remove() end
        if crossHor ~= nil or crossVer ~= nil then
            return
        end

        local self = {
            Visible = true;
            Transparency = (1 - crosshair.transparency);
            Thickness = crosshair.thickness;
            Color = crosshair.color;
        }

        if crosshair.enabled ~= true then
            self.Visible = false
        end
        local h,v = newdrawing("Line", self), newdrawing("Line", self)

        if self.Visible then
            local vpSize = camera.ViewportSize/2
            local size = crosshair.size/2
            local x,y = vpSize.X, vpSize.Y

            h.From = v2new(x - size, y)
            h.To = v2new(x + size, y)
            
            v.From = v2new(x, y - size)
            v.To = v2new(x, y + size)
        end

        crossHor = h
        crossVer = v
    end

    local function updateCrosshair() -- no reason at all to update this each frame
        -- I will replace with ViewportSize.Changed later
        if completeStop then return end

        if crossHor == nil or crossVer == nil then
            return drawCrosshair()
        end

        local visible = crosshair.enabled

        crossHor.Visible = visible
        crossVer.Visible = visible

        if visible then
            local vpSize = camera.ViewportSize / 2
            local size = crosshair.size/2
            local x,y = vpSize.X, vpSize.Y

            local color = crosshair.color
            crossHor.Color = color
            crossVer.Color = color
            
            local trans = (1 - crosshair.transparency)
            crossHor.Transparency = trans
            crossVer.Transparency = trans

            local thick = crosshair.thickness
            crossHor.Thickness = thick
            crossVer.Thickness = thick

            crossHor.From = v2new(x - size, y)
            crossHor.To = v2new(x + size, y)
        
            crossVer.From = v2new(x, y - size)
            crossVer.To = v2new(x, y + size)
        end
    end

    function crosshair:Remove()
        if crossHor ~= nil then
            crossHor:Remove()
            crossHor = nil
        end
        if crossVer ~= nil then
            crossVer:Remove()
            crossVer = nil
        end
    end

    function crosshair:End()
        completeStop = true
        if crossHor ~= nil then
            crossHor:Remove()
            crossHor = nil
        end
        if crossVer ~= nil then
            crossVer:Remove()
            crossVer = nil
        end
        crosshair.enabled = false
    end

    crosshair.step = updateCrosshair
    --function crosshair.step()
    --    updateCrosshair()        
    --end
end


do
    --/ Tracers

    local tracers_settings = {}
    tracers_settings.enabled = settings:Get("tracers.enabled", true)
    tracers_settings.origin = v2new(camera.ViewportSize.X/2, camera.ViewportSize.Y)
    tracers_settings.frommouse = settings:Get("tracers.frommouse", true)
    tracers_settings.transparency = .6
    tracers_settings.thickness = 1.5
    tracers_settings.showteam = settings:Get("tracers.showteam", true)

    tracers_settings.drawdistance = settings:Get("tracers.drawdistance", 4000)
    tracers_settings.showvisible = settings:Get("tracers.showvisible", true)

    tracers_settings.enemycolor = Color3.fromRGB(255,7,58) -- 238,38,37, 255,0,13, 255,7,58
    tracers_settings.teamcolor = Color3.fromRGB(121,255,97) -- 121,255,97, 57,255,20
    tracers_settings.visiblecolor = Color3.fromRGB(0, 141, 255)

    setmetatable(tracers, {
        __index = function(self, index)
            if tracers_settings[index] ~= nil then
                local Value = tracers_settings[index]
                if typeof(Value) == "table" then
                    return typeof(Value) == "table" and Value.Value
                else
                    return Value
                end
            end
            warn(("AH8_ERROR : TracersSettings : Tried to index %s"):format(tostring(index)))
        end;
        __newindex = function(self, index, value)
            if typeof(value) ~= "function" then
                if tracers_settings[index] then
                    local v = tracers_settings[index]
                    if typeof(v) ~= "table" then
                        tracers_settings[index] = value
                        return
                    elseif v.Set then
                        v:Set(value)
                        return
                    end
                end
            end
            rawset(self, index, value)
        end;
    })

    local worldToViewportPoint = camera.WorldToViewportPoint

    local completeStop = false
    local drawn = {}

    local function drawTemplate(player)
        if completeStop then return end

        if drawn[player] then
            return drawn[player]
           --tracers:Remove(player)
        end


        local a = newdrawing("Line", {
            Color = tracers.enemycolor;
            Thickness = tracers.thickness;
            Transparency = 1 - tracers.transparency;
            Visible = false;
        })
        drawn[player] = a
        return a
    end

    function tracers:Draw(player, character, root, humanoid, onscreen, isteam, dist, screenpos)
        if completeStop then return end

        if tracers.enabled ~= true then return tracers:Remove(player) end
        if character == nil then return tracers:Remove(player) end

        if tracers.showteam~=true and isteam then return tracers:Remove(player) end

        if root == nil then return tracers:Remove(player) end

        if dist then
            if dist > tracers.drawdistance then
                return tracers:Remove(player)
            end
        end

        local screenpos = worldToViewportPoint(camera, root.Position)

        local line
        if drawn[player] ~= nil then
            line = drawn[player]
        elseif onscreen then
            line = drawTemplate(player)
        end
        if line then
            if onscreen then
                line.From = tracers.origin
                line.To = v2new(screenpos.X, screenpos.Y)
        
                local color
                if isteam == false and tracers.showvisible then
                    if utility.isvisible(character, root, 0) then
                        color = tracers.visiblecolor
                    else
                        color = isteam and tracers.teamcolor or tracers.enemycolor
                    end
                else
                    color = isteam and tracers.teamcolor or tracers.enemycolor
                end

                line.Color = color
            end
            line.Visible = onscreen
        end
        --return line
    end

    function tracers:Hide(player)
        if completeStop then return end

        local line = drawn[player]
        if line then
            line.Visible = false
        end
    end

    function tracers:Remove(player)
        if drawn[player] ~= nil then
            drawn[player]:Remove()
            drawn[player] = nil
        end
    end

    function tracers:RemoveAll()
        for i,v in pairs(drawn) do
            pcall(function()
                v:Remove()
            end)
            drawn[i] = nil
        end
        drawn = {}
    end
    function tracers:End()
        completeStop = true
        for i,v in pairs(drawn) do
            pcall(function()
                v:Remove()
            end)
            drawn[i] = nil
        end
        drawn = {}
    end
end

do
    --/ ESP
    local esp_settings = {}

    esp_settings.enabled = settings:Get("esp.enabled", true)
    esp_settings.showteam = settings:Get("esp.showteam", true)
    
    esp_settings.teamcolor = Color3.fromRGB(57,255,20) -- 121,255,97, 57,255,20
    esp_settings.enemycolor = Color3.fromRGB(255,7,58) -- 238,38,37, 255,0,13, 255,7,58
    esp_settings.visiblecolor = Color3.fromRGB(0, 141, 255) -- 0, 141, 255


    esp_settings.size = settings:Get("esp.size", 16)
    esp_settings.centertext = settings:Get("esp.centertext", true)
    esp_settings.outline = settings:Get("esp.outline", true)
    esp_settings.transparency = settings:Get("esp.transparency", 0.1)

    esp_settings.drawdistance = settings:Get("esp.drawdistance", 1500)


    esp_settings.showvisible = settings:Get("esp.showvisible", true)

    esp_settings.yoffset = settings:Get("esp.yoffset", 0)

    esp_settings.showhealth = settings:Get("esp.showhealth", true)
    esp_settings.showdistance = settings:Get("esp.showdistance", true)


    setmetatable(esp, {
        __index = function(self, index)
            if esp_settings[index] ~= nil then
                local Value = esp_settings[index]
                if typeof(Value) == "table" then
                    return typeof(Value) == "table" and Value.Value
                else
                    return Value
                end
            end
            warn(("AH8_ERROR : EspSettings : Tried to index %s"):format(tostring(index)))
        end;
        __newindex = function(self, index, value)
            if typeof(value) ~= "function" then
                if esp_settings[index] then
                    local v = esp_settings[index]
                    if typeof(v) ~= "table" then
                        esp_settings[index] = value
                        return
                    elseif v.Set then
                        v:Set(value)
                        return
                    end
                end
            end
            rawset(self, index, value)
        end;
    })

    local unpack = unpack
    local findFirstChild = Instance.new("Part").FindFirstChild
    local worldToViewportPoint = camera.WorldToViewportPoint
    local getBoundingBox = Instance.new("Model").GetBoundingBox
    local getExtentsSize = Instance.new("Model").GetExtentsSize

    local floor = math.floor
    local insert = table.insert
    local concat = table.concat

    local drawn = {}
    local completeStop = false

    local function drawTemplate(player)
        if completeStop then return end
        if drawn[player] then return drawn[player] end

        local obj = newdrawing("Text", {
            Text = "n/a",
            Size = esp.size,
            Color = esp.enemycolor,
            Center = esp.centertext,
            Outline = esp.outline,
            Transparency = (1 - esp.transparency),
        })
        return obj
    end

    function esp:Draw(player, character, root, humanoid, onscreen, isteam, dist)
        if completeStop then return end
        if character == nil then return esp:Remove(player) end
        if root == nil then return esp:Remove(player) end
        if esp.showteam~=true and isteam then return esp:Remove(player) end

        if dist then
            if dist > esp.drawdistance then
                return esp:Remove(player)
            end
        end

        local where, isvis = worldToViewportPoint(camera, (root.CFrame * esp.offset).p);
        --if not isvis then return esp:Remove(player) end


        local oesp = drawn[player]
        if oesp == nil then
            oesp = drawTemplate(player)
            drawn[player] = oesp
        end
        
        if oesp then
            oesp.Visible = isvis
            if isvis then
                oesp.Position = v2new(where.X, where.Y)

                local color
                if isteam == false and esp.showvisible then
                    if utility.isvisible(character, root, 0) then
                        color = esp.visiblecolor
                    else
                        color = isteam and esp.teamcolor or esp.enemycolor
                    end
                else
                    color = isteam and esp.teamcolor or esp.enemycolor
                end

                oesp.Color = color

                oesp.Center = esp.centertext
                oesp.Size = esp.size
                oesp.Outline = esp.outline
                oesp.Transparency = (1 - esp.transparency)

                local texts = {
                    player.Name,
                }
                
                local b = humanoid and esp.showhealth and ("%s/%s"):format(floor(humanoid.Health + .5), floor(humanoid.MaxHealth + .5))
                if b then
                    insert(texts, b)
                end
                local c = dist and esp.showdistance and ("%s"):format(floor(dist + .5))
                if c then
                    insert(texts, c)
                end

                local text = "[  " .. concat(texts, " | ") .. " ]"
                oesp.Text = text
            end
        end
    end

    function esp:Remove(player)
        local data = drawn[player]
        if data ~= nil then
            data:Remove()
            drawn[player] = nil
        end
    end

    function esp:RemoveAll()
        for i,v in pairs(drawn) do
            pcall(function() v:Remove() end)
            drawn[i] = nil
        end
    end

    function esp:End()
        completeStop = true
        esp:RemoveAll()
    end
end


do
    --/ Boxes

    local boxes_settings = {}
    boxes_settings.enabled = settings:Get("boxes.enabled", true)
    boxes_settings.transparency = settings:Get("boxes.transparency", .2)
    boxes_settings.thickness = settings:Get("boxes.thickness", 1.5)
    boxes_settings.showteam = settings:Get("boxes.showteam", true)

    boxes_settings.teamcolor = Color3.fromRGB(57,255,20) -- 121,255,97,  57,255,20
    boxes_settings.enemycolor = Color3.fromRGB(255,7,58) -- 238,38,37, 255,0,13, 255,7,58
    boxes_settings.visiblecolor = Color3.fromRGB(0, 141, 255)

    boxes_settings.thirddimension = settings:Get("boxes.thirddimension", false)

    boxes_settings.showvisible = settings:Get("boxes.showvisible", true)

    boxes_settings.dist3d = settings:Get("boxes.dist3d", 1000)
    boxes_settings.drawdistance = settings:Get("boxes.drawdistance", 4000)
    boxes_settings.color = Color3.fromRGB(255, 50, 50)

    setmetatable(boxes, {
        __index = function(self, index)
            if boxes_settings[index] ~= nil then
                local Value = boxes_settings[index]
                if typeof(Value) == "table" then
                    return typeof(Value) == "table" and Value.Value
                else
                    return Value
                end
            end
            warn(("AH8_ERROR : BoxesSettings : Tried to index %s"):format(tostring(index)))
        end;
        __newindex = function(self, index, value)
            if typeof(value) ~= "function" then
                if boxes_settings[index] then
                    local v = boxes_settings[index]
                    if typeof(v) ~= "table" then
                        boxes_settings[index] = value
                        return
                    elseif v.Set then
                        v:Set(value)
                        return
                    end
                end
            end
            rawset(self, index, value)
        end;
    })

    local unpack = unpack
    local findFirstChild = Instance.new("Part").FindFirstChild
    local worldToViewportPoint = camera.WorldToViewportPoint
    local worldToScreenPoint = camera.WorldToScreenPoint
    local getBoundingBox = Instance.new("Model").GetBoundingBox
    local getExtentsSize = Instance.new("Model").GetExtentsSize

    local completeStop = false
    local drawn = {}
    local function drawTemplate(player, amount)
        if completeStop then return end

        if drawn[player] then
            if #drawn[player] == amount then
                return drawn[player]
            end
            boxes:Remove(player)
        end

        local props = {
            Visible = true;
            Transparency = 1 - boxes.transparency;
            Thickness = boxes.thickness;
            Color = boxes.color;
        }

        local a = {}
        for i = 1,amount or 4 do
            a[i] = newdrawing("Line", props)
        end

        drawn[player] = {unpack(a)}
        return unpack(a)
    end

    local function updateLine(line, from, to, vis, color)
        if line == nil then return end

        line.Visible = vis
        if vis then
            line.From = from
            line.To = to
            line.Color = color
        end
    end

    function boxes:Draw(player, character, root, humanoid, onscreen, isteam, dist) -- No skid plox
        if completeStop then return end
        if character == nil then return boxes:Remove(player) end
        if root == nil then return boxes:Remove(player) end
        if not onscreen then return boxes:Remove(player) end
        if boxes.showteam == false and isteam then return boxes:Remove(player) end

        local _3dimension = boxes.thirddimension
        if dist ~= nil then
            if dist > boxes.drawdistance then
                return boxes:Remove(player)
            elseif _3dimension and dist > boxes.dist3d then
                _3dimension = false
            end
        end

        local color
        if isteam == false and boxes.showvisible then
            if utility.isvisible(character, root, 0) then
                color = boxes.visiblecolor
            else
                color = isteam and boxes.teamcolor or boxes.enemycolor
            end
        else
            color = isteam and boxes.teamcolor or boxes.enemycolor
        end

        --size = ... lastsize--, v3new(5,8,0) --getBoundingBox(character)--]] root.CFrame, getExtentsSize(character)--]] -- Might change this later idk + idc
        if _3dimension then

            local tlb, trb, blb, brb, tlf, trf, blf, brf, tlf0, trf0, blf0, brf0
            if drawn[player] == nil or #drawn[player] ~= 12 then
                tlb, trb, blb, brb, tlf, trf ,blf, brf, tlf0, trf0, blf0, brf0 = drawTemplate(player, 12)
            else
                tlb, trb, blb, brb, tlf, trf ,blf, brf, tlf0, trf0, blf0, brf0 = unpack(drawn[player])
            end

            local pos, size = root.CFrame, root.Size--lastsize--, v3new(5,8,0)

            local topleftback, topleftbackvisible = worldToViewportPoint(camera, (pos * cfnew(-size.X, size.Y, size.Z)).p);
            local toprightback, toprightbackvisible = worldToViewportPoint(camera, (pos * cfnew(size.X, size.Y, size.Z)).p);
            local btmleftback, btmleftbackvisible = worldToViewportPoint(camera, (pos * cfnew(-size.X, -size.Y, size.Z)).p);
            local btmrightback, btmrightbackvisible = worldToViewportPoint(camera, (pos * cfnew(size.X, -size.Y, size.Z)).p);

            local topleftfront, topleftfrontvisible = worldToViewportPoint(camera, (pos * cfnew(-size.X, size.Y, -size.Z)).p);
            local toprightfront, toprightfrontvisible = worldToViewportPoint(camera, (pos * cfnew(size.X, size.Y, -size.Z)).p);
            local btmleftfront, btmleftfrontvisible = worldToViewportPoint(camera, (pos * cfnew(-size.X, -size.Y, -size.Z)).p);
            local btmrightfront, btmrightfrontvisible = worldToViewportPoint(camera, (pos * cfnew(size.X, -size.Y, -size.Z)).p);

            local topleftback = v2new(topleftback.X, topleftback.Y)
            local toprightback = v2new(toprightback.X, toprightback.Y)
            local btmleftback = v2new(btmleftback.X, btmleftback.Y)
            local btmrightback = v2new(btmrightback.X, btmrightback.Y)

            local topleftfront = v2new(topleftfront.X, topleftfront.Y)
            local toprightfront = v2new(toprightfront.X, toprightfront.Y)
            local btmleftfront = v2new(btmleftfront.X, btmleftfront.Y)
            local btmrightfront = v2new(btmrightfront.X, btmrightfront.Y)

            -- pls don't copy this bad code
			updateLine(tlb, topleftback, toprightback, topleftbackvisible, color)
            updateLine(trb, toprightback, btmrightback, toprightbackvisible, color)
            updateLine(blb, btmleftback, topleftback, btmleftbackvisible, color)
            updateLine(brb, btmleftback, btmrightback, btmrightbackvisible, color)

            --

            updateLine(brf, btmrightfront, btmleftfront, btmrightfrontvisible, color)
            updateLine(tlf, topleftfront, toprightfront, topleftfrontvisible, color)
            updateLine(trf, toprightfront, btmrightfront, toprightfrontvisible, color)
            updateLine(blf, btmleftfront, topleftfront, btmleftfrontvisible, color)

            --

            updateLine(brf0, btmrightfront, btmrightback, btmrightfrontvisible, color)
            updateLine(tlf0, topleftfront, topleftback, topleftfrontvisible, color)
            updateLine(trf0, toprightfront, toprightback, toprightfrontvisible, color)
            updateLine(blf0, btmleftfront, btmleftback, btmleftfrontvisible, color)
            return
        else

            local tl, tr, bl, br
            if drawn[player] == nil or #drawn[player] ~= 4 then
                tl, tr, bl, br = drawTemplate(player, 4)
            else
                tl, tr, bl, br = unpack(drawn[player])
            end

            local pos, size = root.CFrame, root.Size

            local topleft, topleftvisible = worldToViewportPoint(camera, (pos * cfnew(-size.X, size.Y, 0)).p);
            local topright, toprightvisible = worldToViewportPoint(camera, (pos * cfnew(size.X, size.Y, 0)).p);
            local btmleft, btmleftvisible = worldToViewportPoint(camera, (pos * cfnew(-size.X, -size.Y, 0)).p);
            local btmright, btmrightvisible = worldToViewportPoint(camera, (pos * cfnew(size.X, -size.Y, 0)).p);

            local topleft = v2new(topleft.X, topleft.Y)
            local topright = v2new(topright.X, topright.Y)
            local btmleft = v2new(btmleft.X, btmleft.Y)
            local btmright = v2new(btmright.X, btmright.Y)

            updateLine(tl, topleft, topright, topleftvisible, color)
            updateLine(tr, topright, btmright, toprightvisible, color)
            updateLine(bl, btmleft, topleft, btmleftvisible, color)
            updateLine(br, btmleft, btmright, btmrightvisible, color)
            return
        end


        -- I have never been more bored when doing 3d boxes.
    end

    function boxes:Remove(player)
        local data = drawn[player]
        if data == nil then return end

        if data then
            for i,v in pairs(data) do
                v:Remove()
                data[i] = nil
            end
        end
        drawn[player] = nil
    end

    function boxes:RemoveAll()
        for i,v in pairs(drawn) do
            pcall(function()
                for i2,v2 in pairs(v) do
                    v2:Remove()
                    v[i] = nil
                end
            end)
            drawn[i] = nil
        end
        drawn = {}
    end

    function boxes:End()
        completeStop = true
        for i,v in pairs(drawn) do
            for i2,v2 in pairs(v) do
                pcall(function()
                    v2:Remove()
                    v[i2] = nil
                end)
            end
            drawn[i] = nil
        end
        drawn = {}
    end
end


do
    --/ Visuals

    visuals.enabled = settings:Get("visuals.enabled", true)

    local getPlayers = players.GetPlayers

    local credits
    local circle

    local completeStop = false
    bindEvent(players.PlayerRemoving, function(p)
        if completeStop then return end
        tracers:Remove(p)
        boxes:Remove(p)
        esp:Remove(p)
    end)

    local profilebegin = DEBUG_MODE and debug.profilebegin or function() end
    local profileend = DEBUG_MODE and debug.profileend or function() end


    local unpack = unpack
    local findFirstChild = Instance.new("Part").FindFirstChild
    local worldToViewportPoint = camera.WorldToViewportPoint

    local function remove(p)
        esp:Remove(p)
        boxes:Remove(p)
        tracers:Remove(p)
    end

    local hashes = {}
    function visuals.step()
        --if visuals.enabled ~= true then return clearDrawn() end
        if completeStop then return end


        local viewportsize = camera.ViewportSize
        if credits == nil then
            credits = newdrawing("Text", {
                Text = "AimBot v1, Tech#1182"; -- yes now be happy this is free
                Color = Color3.new(255,255,255);
                Size = 25.0;
                Transparency = .8;
                Position = v2new(viewportsize.X/8, 6);
                Outline = true;
                Visible = true;
            })
        else
            credits.Position = v2new(viewportsize.X/8, 6);
        end

        if aimbot.enabled and aimbot.fovenabled and visuals.enabled then
            profilebegin("fov.step")
            if circle == nil then
                circle = newdrawing("Circle", {
                    Position = v2new(mouse.X, mouse.Y+36),
                    Radius = aimbot.fovsize,
                    Color = Color3.fromRGB(240,240,240),
                    Thickness = aimbot.fovthickness,
                    Filled = false,
                    Transparency = .8,
                    NumSides = aimbot.fovsides,
                    Visible = aimbot.fovshow;
                })
            else
                if aimbot.fovshow then                    
                    circle.Position = v2new(mouse.X, mouse.Y+36)
                    circle.Radius = aimbot.fovsize
                    circle.NumSides = aimbot.fovsides
                    circle.Thickness = aimbot.fovthickness
                end
                circle.Visible = aimbot.fovshow
            end
            profileend("fov.step")
        elseif circle ~= nil then
            circle:Remove()
            circle = nil
        end

        if visuals.enabled and crosshair.enabled then
            profilebegin("crosshair.step")
            crosshair.step()
            profileend("crosshair.step")
        else
            crosshair:Remove()
        end

        if visuals.enabled and (esp.enabled or boxes.enabled or tracers.enabled) then
            profilebegin("tracers.origin")
            if tracers.frommouse then 
                tracers.origin = v2new(mouse.X, mouse.Y+36) -- thanks roblox
            else
                tracers.origin = v2new(viewportsize.X/2, viewportsize.Y)
            end
            profileend("tracers.origin")

            if esp.enabled then
                esp.offset = cfnew(0, esp.yoffset, 0)
            end

            for i,v in pairs(getPlayers(players)) do
                if (v~=locpl) then
                    local character = utility.getcharacter(v)
                    if character and isDescendantOf(character, game) == true then
                        local root = utility.getroot(character)
                        local humanoid = findFirstChildOfClass(character, "Humanoid")
                        if root and isDescendantOf(character, game) == true then
                            local screenpos, onscreen = worldToViewportPoint(camera, root.Position)
                            local dist = utility.myroot and (utility.myroot.Position - root.Position).Magnitude
                            local isteam = (v.Team~=nil and v.Team==locpl.Team) and not v.Neutral or false

                            if boxes.enabled then -- Profilebegin is life
                                profilebegin("boxes.draw")
                                boxes:Draw(v, character, root, humanoid, onscreen, isteam, dist)
                                profileend("boxes.draw")
                            else
                                boxes:Remove(v)
                            end
                            if tracers.enabled then
                                profilebegin("tracers.draw")
                                tracers:Draw(v, character, root, humanoid, onscreen, isteam, dist, screenpos)
                                profileend("tracers.draw")
                            else
                                tracers:Remove(v)
                            end
        
                            if esp.enabled then
                                profilebegin("esp.draw")
                                esp:Draw(v, character, root, humanoid, onscreen, isteam, dist)
                                profileend("esp.draw")
                            else
                                esp:Remove(v)
                            end
                        else
                            remove(v)
                        end
                    else
                        remove(v)
                    end
                end
            end
        else
            -- mhm
            tracers:RemoveAll()
            boxes:RemoveAll()
            esp:RemoveAll()
            crosshair:Remove()
        end
    end

    function visuals:End()
        completeStop = true
        crosshair:End()
        boxes:End()
        tracers:End()
        esp:End()

        clearDrawn()
    end

    spawn(function()
        while ah8 and ah8.enabled do -- I dont know why I am doing this
            for i,v in pairs(hashes) do
                hashes[i] = nil
                wait()
            end
            wait(3)
        end
    end)
end



-- Ok yes
do
    --/ Run

    local pcall = pcall;
    local tostring = tostring;
    local warn = warn;
    local debug = debug;
    local profilebegin = DEBUG_MODE and debug.profilebegin or function() end
    local profileend = DEBUG_MODE and debug.profileend or function() end

    local renderstep = runservice.RenderStepped
    local heartbeat = runservice.Heartbeat
    local stepped = runservice.Stepped
    local wait = renderstep.wait

    local function Warn(a, ...) -- ok frosty get to bed
        warn(tostring(a):format(...))
    end
    
    run.dt = 0
    run.time = tick()

    local engine = {
        {
            name = 'visuals.step',
            func = visuals.step
        };
    }
    local heartengine = {
        {
            name = 'aimbot.step',
            func = aimbot.step
        };
    }
    local whilerender = {
    }

    run.onstep = {}
    run.onthink = {}
    run.onrender = {}
    function run.wait()
        wait(renderstep)
    end

    local fireonstep = event.new(run.onstep)
    local fireonthink = event.new(run.onthink)
    local fireonrender = event.new(run.onrender)

    local rstname = "AH.Renderstep"
    bindEvent(renderstep, function(delta)
        profilebegin(rstname)
        local ntime = tick()
        run.dt = ntime - run.time
        run.time = ntime

        for i,v in pairs(engine) do

            profilebegin(v.name)
            xpcall(v.func, function(err)
                if (DEBUG_MODE == true) then
                    Warn("AH8_ERROR (RENDERSTEPPED) : Failed to run %s! %s | %s", v.name, tostring(err), debug.traceback())
                end
                engine[i] = nil
            end, run.dt)
            profileend(v.name)

        end

        profileend(rstname)
    end)

    local hbtname = "AH.Heartbeat"
    bindEvent(heartbeat, function(delta)
        profilebegin(hbtname)

        for i,v in pairs(heartengine) do

            profilebegin(v.name)
            xpcall(v.func, function(err)
                if (DEBUG_MODE == true) then
                    Warn("AH8_ERROR (HEARTBEAT) : Failed to run %s! %s | %s", v.name, tostring(err), debug.traceback())
                end
                heartengine[i] = nil
            end, delta)
            profileend(v.name)

        end

        profileend(hbtname)
    end)

    local stpname = "AH.Stepped"
    bindEvent(stepped, function(delta)
        
        profilebegin(stpname)

        for i,v in pairs(whilerender) do

            profilebegin(v.name)
            xpcall(v.func, function(err)
                if (DEBUG_MODE == true) then
                    Warn("AH8_ERROR (STEPPED) : Failed to run %s! %s | %s", v.name, tostring(err), debug.traceback())
                end
                heartengine[i] = nil
            end, delta)
            profileend(v.name)

        end

        profileend(stpname)
    end)
end

do
    --/ Main or something I am not sure what I am writing anymore
    settings:Save()

    ah8.enabled = true
    function ah8:close()
        spawn(function() pcall(visuals.End, visuals) end)
        spawn(function() pcall(aimbot.End, aimbot) end)
        spawn(function() pcall(hud.End, hud) end)
        spawn(function()
            for i,v in pairs(connections) do
                pcall(function() v:Disconnect() end)
            end
        end)
        ah8 = nil
        shared.ah8 = nil -- k

        settings:Save()
    end

    ah8.visible = hud.Visible
    function ah8:show()
        hud:show()
        ah8.visible = hud.Visible
    end

    function ah8:hide()
        hud:hide()
        ah8.visible = hud.Visible
    end

    setmetatable(ah8, { -- ok safazi be happy now
        __newindex = function(self, index, value)
            if (index == "Keybind") then
                settings:Set("hud.keybind", value)
                hud.Keybind = value
                return
            end
        end;
    })

    shared.ah8 = ah8

    local players = game:GetService("Players")
    local loc = players.LocalPlayer
    bindEvent(players.PlayerRemoving, function(p)
        if p == loc then
            settings:Save()
        end
    end)

end


-- I didn't think this ui lib through
local Aiming = hud:AddTab({
	Text = "Aiming",
})


local AimbotToggle = Aiming:AddToggleCategory({
	Text = "Aimbot",
	State = aimbot.enabled,
}, function(state) 
    aimbot.enabled = state
end)


AimbotToggle:AddKeybind({
	Text = "keybind",
	Current = aimbot.keybind,
}, function(new)
    aimbot.keybind = new.Name 
end)

 
AimbotToggle:AddToggle({
	Text = "Press To Enable",
	State = aimbot.presstoenable,
}, function(state) 
    aimbot.presstoenable = state
end)

AimbotToggle:AddToggle({
	Text = "Lock To Target",
	State = aimbot.locktotarget,
}, function(state) 
    aimbot.locktotarget = state
end)


AimbotToggle:AddToggle({
	Text = "Check If Alive",
	State = aimbot.checkifalive,
}, function(state) 
    aimbot.checkifalive = state
end)

-- settings stuff
local AimbotSettings = Aiming:AddCategory({
	Text = "Settings",
})

AimbotSettings:AddLabel({
    Text = "decrease sens if aimbot is wobbly"
})

AimbotSettings:AddSlider({
    Text = "Sensitivity",
    Current = aimbot.sensitivity
}, {0.01, 10, 0.01}, function(new) 
    aimbot.sensitivity = new
end)

AimbotSettings:AddToggle({
    Text = "Ignore Team",
    State = aimbot.ignoreteam
}, function(new)
    aimbot.ignoreteam = new
end)


AimbotSettings:AddToggle({
    Text = "Ignore Walls",
    State = aimbot.ignorewalls
}, function(new)
    aimbot.ignorewalls = new
end)

AimbotSettings:AddSlider({
    Text = "Max Obscuring Parts",
    Current = aimbot.maxobscuringparts,
}, {0, 50, 1}, function(new)
    aimbot.maxobscuringparts = new
end)



local FieldOfView = Aiming:AddToggleCategory({
    Text = "fov",
    State = aimbot.fovenabled,
}, function(state) 
    aimbot.fovenabled = state
end)

FieldOfView:AddSlider({
    Text = "Radius",
    Current = aimbot.fovsize,
}, {1, 1000, 1}, function(new)
    aimbot.fovsize = new
end)

FieldOfView:AddSlider({
    Text = "Sides",
    Current = aimbot.fovsides,
}, {6, 40, 1}, function(new)
    aimbot.fovsides = new
end)


FieldOfView:AddSlider({
    Text = "Thickness",
    Current = aimbot.fovthickness,
}, {0.1, 50, 0.1}, function(new)
    aimbot.fovthickness = new
end)



local Visuals = hud:AddTab({
    Text = "Visuals"
})

Visuals:AddToggle({
    Text = "Enabled",
    State = visuals.enabled,
}, function(new)
    visuals.enabled = new
end)

local Boxes = Visuals:AddToggleCategory({
    Text = "Boxes",
    State = boxes.enabled,
}, function(new)
    boxes.enabled = new
end)

Boxes:AddToggle({
    Text = "Show Team",
    State = boxes.showteam,
}, function(new)
    boxes.showteam = new
end)

Boxes:AddToggle({
    Text = "Visible check",
    State = boxes.showvisible,
}, function(new)
    boxes.showvisible = new
end)

Boxes:AddSlider({
    Text = "Draw Distance",
    Current = boxes.drawdistance,
}, {100,100000,100}, function(new)
    boxes.drawdistance = new
end)

Boxes:AddToggle({
    Text = "3d",
    State = boxes.thirddimension,
}, function(new)
    boxes.thirddimension = new
end)

Boxes:AddSlider({
    Text = "3d distance",
    Current = boxes.dist3d,
}, {5,10000,5}, function(new)
    boxes.dist3d = new
end)


local Esp = Visuals:AddToggleCategory({
    Text = "Esp",
    State = esp.enabled,
}, function(new)
    esp.enabled = new
end)

Esp:AddToggle({
    Text = "Show Team",
    State = esp.showteam
}, function(new)
    esp.showteam = new
end)

Esp:AddToggle({
    Text = "Visible check",
    State = esp.showvisible,
}, function(new)
    esp.showvisible = new
end)

Esp:AddSlider({
    Text = "Offset",
    Current = esp.yoffset,
}, {-50, 50, 0.01}, function(new)
    esp.yoffset = new
end)

Esp:AddSlider({
    Text = "Transparency",
    Current = esp.transparency
}, {0, 1, 0.01}, function(new)
    esp.transparency = new
end)

Esp:AddSlider({
    Text = "Size",
    Current = esp.size,
}, {1, 100, 1}, function(new)
    esp.size = new
end)

Esp:AddToggle({
    Text = "Center Text",
    State = esp.centertext
}, function(new)
    esp.centertext = new
end)

Esp:AddToggle({
    Text = "Outline",
    State = esp.outline,
}, function(new)
    esp.outline = new
end)

Esp:AddSlider({
    Text = "Draw Distance",
    Current = esp.drawdistance
}, {100,100000,100}, function(new)
    esp.drawdistance = new
end)


--
local Tracers = Visuals:AddToggleCategory({
    Text = "Tracers",
    State = tracers.enabled,
}, function(new)
    tracers.enabled = new
end)

Tracers:AddToggle({
    Text = "Show Team",
    State = tracers.showteam
}, function(new)
    tracers.showteam = new
end)

Tracers:AddToggle({
    Text = "Visible check",
    State = tracers.showvisible,
}, function(new)
    tracers.showvisible = new
end)

Tracers:AddToggle({
    Text = "From Mouse",
    State = tracers.frommouse,
}, function(new)
    tracers.frommouse = new
end)

Tracers:AddSlider({
    Text = "Draw Distance",
    Current = tracers.drawdistance,
}, {100,100000,100}, function(new)
    tracers.drawdistance = new
end)


local Crosshair = Visuals:AddToggleCategory({
    Text = "Crosshair",
    State = crosshair.enabled,
}, function(new)
    crosshair.enabled = new
end)

Crosshair:AddSlider({
    Text = "Transparency",
    Current = crosshair.transparency
}, {0,1,0.01}, function(new)
    crosshair.transparency = new
end)

Crosshair:AddSlider({
    Text = "Size",
    Current = crosshair.size,
}, {1,2000,1}, function(new)
    crosshair.size = new
end)

Crosshair:AddSlider({
    Text = "Thickness",
    Current = crosshair.thickness
}, {1,50,1}, function(new)
    crosshair.thickness = new
end)


local Hud = hud:AddTab({
    Text = "Hud",
})

hud.Keybind = settings:Get("hud.keybind", "RightAlt").Value
Hud:AddKeybind({
    Text = "Toggle",
    Current = hud.Keybind,
}, function(new)
    settings:Set("hud.keybind", new.Name)
    hud.Keybind = new.Name
end)

Hud:AddButton({
    Text = "Exit"
}, function()
    ah8:close()
end)

warn("AH8_MAIN : Reached end of script")
  	end    
})

--[[
Name = <string> - The name of the button.
Callback = <function> - The function of the button.
]]

Tab:AddButton({
	Name = "ESP!",
	Callback = function()
      		-- Technical ESP

assert(Drawing, 'exploit not supported')

local UserInputService = game:GetService'UserInputService';
local HttpService = game:GetService'HttpService';
local GUIService = game:GetService'GuiService';
local RunService = game:GetService'RunService';
local Players = game:GetService'Players';
local LocalPlayer = Players.LocalPlayer;
local Camera = workspace.CurrentCamera
local Mouse = LocalPlayer:GetMouse();
local Menu = {};
local MouseHeld = false;
local LastRefresh = 0;
local OptionsFile = 'IC3_ESP_SETTINGS.dat';
local Binding = false;
local BindedKey = nil;
local OIndex = 0;
local LineBox = {};
local UIButtons = {};
local Sliders = {};
local Dragging = false;
local DraggingUI = false;
local DragOffset = Vector2.new();
local DraggingWhat = nil;
local OldData = {};
local IgnoreList = {};
local Red = Color3.new(1, 0, 0);
local Green = Color3.new(0, 1, 0);
local MenuLoaded = false;

shared.MenuDrawingData = shared.MenuDrawingData or { Instances = {} };
shared.PlayerData = shared.PlayerData or {};
shared.RSName = shared.RSName or ('UnnamedESP_by_ic3-' .. HttpService:GenerateGUID(false));

local GetDataName = shared.RSName .. '-GetData';
local UpdateName = shared.RSName .. '-Update';

local Debounce = setmetatable({}, {
__index = function(t, i)
return rawget(t, i) or false
end;
});

pcall(function() shared.InputBeganCon:disconnect() end);
pcall(function() shared.InputEndedCon:disconnect() end);

function GetMouseLocation()
return UserInputService:GetMouseLocation();
end

function MouseHoveringOver(Values)
local X1, Y1, X2, Y2 = Values[1], Values[2], Values[3], Values[4]
local MLocation = GetMouseLocation();
return (MLocation.x >= X1 and MLocation.x <= (X1 + (X2 - X1))) and (MLocation.y >= Y1 and MLocation.y <= (Y1 + (Y2 - Y1)));
end

function GetTableData(t) -- basically table.foreach i dont even know why i made this
if typeof(t) ~= 'table' then return end
return setmetatable(t, {
__call = function(t, func)
if typeof(func) ~= 'function' then return end;
for i, v in pairs(t) do
pcall(func, i, v);
end
end;
});
end
local function Format(format, ...)
return string.format(format, ...);
end
function CalculateValue(Min, Max, Percent)
return Min + math.floor(((Max - Min) * Percent) + .5);
end

local Options = setmetatable({}, {
__call = function(t, ...)
local Arguments = {...};
local Name = Arguments[1];
OIndex = OIndex + 1; -- (typeof(Arguments[3]) == 'boolean' and 1 or 0);
rawset(t, Name, setmetatable({
Name = Arguments[1];
Text = Arguments[2];
Value = Arguments[3];
DefaultValue = Arguments[3];
AllArgs = Arguments;
Index = OIndex;
}, {
__call = function(t, v)
if typeof(t.Value) == 'function' then
t.Value();
elseif typeof(t.Value) == 'EnumItem' then
local BT = Menu:GetInstance(Format('%s_BindText', t.Name));
Binding = true;
local Val = 0
while Binding do
wait();
Val = (Val + 1) % 17;
BT.Text = Val <= 8 and '|' or '';
end
t.Value = BindedKey;
BT.Text = tostring(t.Value):match'%w+%.%w+%.(.+)';
BT.Position = t.BasePosition + Vector2.new(t.BaseSize.X - BT.TextBounds.X - 20, -10);
else
local NewValue = v;
if NewValue == nil then NewValue = not t.Value; end
rawset(t, 'Value', NewValue);
if Arguments[2] ~= nil then
if typeof(Arguments[3]) == 'number' then
local AMT = Menu:GetInstance(Format('%s_AmountText', t.Name));
AMT.Text = tostring(t.Value);
AMT.Position = t.BasePosition + Vector2.new(t.BaseSize.X - AMT.TextBounds.X - 10, -10);
else
local Inner = Menu:GetInstance(Format('%s_InnerCircle', t.Name));
Inner.Visible = t.Value;
end
end
end
end;
}));
end;
})

function Load()
local _, Result = pcall(readfile, OptionsFile);
if _ then -- extremely ugly code yea i know but i dont care p.s. i hate pcall
local _, Table = pcall(HttpService.JSONDecode, HttpService, Result);
if _ then
for i, v in pairs(Table) do
if Options[i] ~= nil and Options[i].Value ~= nil and (typeof(Options[i].Value) == 'boolean' or typeof(Options[i].Value) == 'number') then
Options[i].Value = v.Value;
pcall(Options[i], v.Value);
end
end
end
end
end

Options('Enabled', 'ESP Enabled', true);
Options('ShowTeam', 'Show Team', false);
Options('ShowName', 'Show Names', true);
Options('ShowDistance', 'Show Distance', true);
Options('ShowHealth', 'Show Health', true);
Options('ShowBoxes', 'Show Boxes', true);
Options('ShowTracers', 'Show Tracers', true);
Options('ShowDot', 'Show Head Dot', false);
Options('VisCheck', 'Visibility Check', false);
Options('Crosshair', 'Crosshair', false);
Options('TextOutline', 'Text Outline', true);
Options('TextSize', 'Text Size', syn and 18 or 14, 10, 24); -- cuz synapse fonts look weird???
Options('MaxDistance', 'Max Distance', 2500, 100, 5000);
Options('RefreshRate', 'Refresh Rate (ms)', 5, 1, 200);
Options('MenuKey', 'Menu Key', Enum.KeyCode.F4, 1);
Options('ResetSettings', 'Reset Settings', function()
for i, v in pairs(Options) do
if Options[i] ~= nil and Options[i].Value ~= nil and Options[i].Text ~= nil and (typeof(Options[i].Value) == 'boolean' or typeof(Options[i].Value) == 'number') then
Options[i](Options[i].DefaultValue);
end
end
end, 4);
Options('LoadSettings', 'Load Settings', Load, 3);
Options('SaveSettings', 'Save Settings', function()
writefile(OptionsFile, HttpService:JSONEncode(Options));
end, 2)
-- Options.SaveSettings.Value();

Load();

Options('MenuOpen', nil, true);

local function Set(t, i, v)
t[i] = v;
end
local function Combine(...)
local Output = {};
for i, v in pairs{...} do
if typeof(v) == 'table' then
table.foreach(v, function(i, v)
Output[i] = v;
end)
end
end
return Output
end
function IsStringEmpty(String)
if type(String) == 'string' then
return String:match'^%s+$' ~= nil or #String == 0 or String == '' or false;
end
return false
end

function NewDrawing(InstanceName)
local Instance = Drawing.new(InstanceName);
return (function(Properties)
for i, v in pairs(Properties) do
pcall(Set, Instance, i, v);
end
return Instance;
end)
end

function Menu:AddMenuInstace(Name, Instance)
if shared.MenuDrawingData.Instances[Name] ~= nil then
shared.MenuDrawingData.Instances[Name]:Remove();
end
shared.MenuDrawingData.Instances[Name] = Instance;
return Instance;
end
function Menu:UpdateMenuInstance(Name)
local Instance = shared.MenuDrawingData.Instances[Name];
if Instance ~= nil then
return (function(Properties)
for i, v in pairs(Properties) do
-- print(Format('%s %s -> %s', Name, tostring(i), tostring(v)));
pcall(Set, Instance, i, v);
end
return Instance;
end)
end
end
function Menu:GetInstance(Name)
return shared.MenuDrawingData.Instances[Name];
end

function LineBox:Create(Properties)
local Box = { Visible = true }; -- prevent errors not really though dont worry bout the Visible = true thing

local Properties = Combine({
Transparency = 1;
Thickness = 1;
Visible = true;
}, Properties);

Box['TopLeft'] = NewDrawing'Line'(Properties);
Box['TopRight'] = NewDrawing'Line'(Properties);
Box['BottomLeft'] = NewDrawing'Line'(Properties);
Box['BottomRight'] = NewDrawing'Line'(Properties);

function Box:Update(CF, Size, Color, Properties)
if not CF or not Size then return end

local TLPos, Visible1 = Camera:WorldToViewportPoint((CF * CFrame.new( Size.X,  Size.Y, 0)).p);
local TRPos, Visible2 = Camera:WorldToViewportPoint((CF * CFrame.new(-Size.X,  Size.Y, 0)).p);
local BLPos, Visible3 = Camera:WorldToViewportPoint((CF * CFrame.new( Size.X, -Size.Y, 0)).p);
local BRPos, Visible4 = Camera:WorldToViewportPoint((CF * CFrame.new(-Size.X, -Size.Y, 0)).p);
-- ## BEGIN UGLY CODE
if Visible1 then
Box['TopLeft'].Visible = true;
Box['TopLeft'].Color = Color;
Box['TopLeft'].From = Vector2.new(TLPos.X, TLPos.Y);
Box['TopLeft'].To = Vector2.new(TRPos.X, TRPos.Y);
else
Box['TopLeft'].Visible = false;
end
if Visible2 then
Box['TopRight'].Visible = true;
Box['TopRight'].Color = Color;
Box['TopRight'].From = Vector2.new(TRPos.X, TRPos.Y);
Box['TopRight'].To = Vector2.new(BRPos.X, BRPos.Y);
else
Box['TopRight'].Visible = false;
end
if Visible3 then
Box['BottomLeft'].Visible = true;
Box['BottomLeft'].Color = Color;
Box['BottomLeft'].From = Vector2.new(BLPos.X, BLPos.Y);
Box['BottomLeft'].To = Vector2.new(TLPos.X, TLPos.Y);
else
Box['BottomLeft'].Visible = false;
end
if Visible4 then
Box['BottomRight'].Visible = true;
Box['BottomRight'].Color = Color;
Box['BottomRight'].From = Vector2.new(BRPos.X, BRPos.Y);
Box['BottomRight'].To = Vector2.new(BLPos.X, BLPos.Y);
else
Box['BottomRight'].Visible = false;
end
-- ## END UGLY CODE
if Properties then
GetTableData(Properties)(function(i, v)
pcall(Set, Box['TopLeft'], i, v);
pcall(Set, Box['TopRight'], i, v);
pcall(Set, Box['BottomLeft'], i, v);
pcall(Set, Box['BottomRight'], i, v);
end)
end
end
function Box:SetVisible(bool)
pcall(Set, Box['TopLeft'], 'Visible', bool);
pcall(Set, Box['TopRight'], 'Visible', bool);
pcall(Set, Box['BottomLeft'], 'Visible', bool);
pcall(Set, Box['BottomRight'], 'Visible', bool);
end
function Box:Remove()
self:SetVisible(false);
Box['TopLeft']:Remove();
Box['TopRight']:Remove();
Box['BottomLeft']:Remove();
Box['BottomRight']:Remove();
end

return Box;
end

function CreateMenu(NewPosition) -- Create Menu
local function FromHex(HEX)
HEX = HEX:gsub('#', '');
return Color3.fromRGB(tonumber('0x' .. HEX:sub(1, 2)), tonumber('0x' .. HEX:sub(3, 4)), tonumber('0x' .. HEX:sub(5, 6)));
end

local Colors = {
Primary = {
Main = FromHex'424242';
Light = FromHex'6d6d6d';
Dark = FromHex'1b1b1b';
};
Secondary = {
Main = FromHex'e0e0e0';
Light = FromHex'ffffff';
Dark = FromHex'aeaeae';
};
};

MenuLoaded = false;

GetTableData(UIButtons)(function(i, v)
v.Instance.Visible = false;
v.Instance:Remove();
end)
GetTableData(Sliders)(function(i, v)
v.Instance.Visible = false;
v.Instance:Remove();
end)

UIButtons = {};
Sliders = {};

local BaseSize = Vector2.new(300, 580);
local BasePosition = NewPosition or Vector2.new(Camera.ViewportSize.X / 8 - (BaseSize.X / 2), Camera.ViewportSize.Y / 2 - (BaseSize.Y / 2));

Menu:AddMenuInstace('CrosshairX', NewDrawing'Line'{
Visible = false;
Color = Color3.new(0, 1, 0);
Transparency = 1;
Thickness = 1;
});
Menu:AddMenuInstace('CrosshairY', NewDrawing'Line'{
Visible = false;
Color = Color3.new(0, 1, 0);
Transparency = 1;
Thickness = 1;
});

delay(.025, function() -- since zindex doesnt exist
Menu:AddMenuInstace('Main', NewDrawing'Square'{
Size = BaseSize;
Position = BasePosition;
Filled = false;
Color = Colors.Primary.Main;
Thickness = 3;
Visible = true;
});
end);
Menu:AddMenuInstace('TopBar', NewDrawing'Square'{
Position = BasePosition;
Size = Vector2.new(BaseSize.X, 25);
Color = Colors.Primary.Dark;
Filled = true;
Visible = true;
});
Menu:AddMenuInstace('TopBarTwo', NewDrawing'Square'{
Position = BasePosition + Vector2.new(0, 25);
Size = Vector2.new(BaseSize.X, 60);
Color = Colors.Primary.Main;
Filled = true;
Visible = true;
});
Menu:AddMenuInstace('TopBarText', NewDrawing'Text'{
Size = 25;
Position = shared.MenuDrawingData.Instances.TopBarTwo.Position + Vector2.new(25, 15);
Text = 'Unnamed ESP';
Color = Colors.Secondary.Light;
Visible = true;
});
Menu:AddMenuInstace('TopBarTextBR', NewDrawing'Text'{
Size = 15;
Position = shared.MenuDrawingData.Instances.TopBarTwo.Position + Vector2.new(BaseSize.X - 65, 40);
Text = 'by TechHub';
Color = Colors.Secondary.Dark;
Visible = true;
});
Menu:AddMenuInstace('Filling', NewDrawing'Square'{
Size = BaseSize - Vector2.new(0, 85);
Position = BasePosition + Vector2.new(0, 85);
Filled = true;
Color = Colors.Secondary.Main;
Transparency= .5;
Visible = true;
});

local CPos = 0;

GetTableData(Options)(function(i, v)
if typeof(v.Value) == 'boolean' and not IsStringEmpty(v.Text) and v.Text ~= nil then
CPos = CPos + 25;
local BaseSize = Vector2.new(BaseSize.X, 30);
local BasePosition = shared.MenuDrawingData.Instances.Filling.Position + Vector2.new(30, v.Index * 25 - 10);
UIButtons[#UIButtons + 1] = {
Option = v;
Instance = Menu:AddMenuInstace(Format('%s_Hitbox', v.Name), NewDrawing'Square'{
Position = BasePosition - Vector2.new(30, 15);
Size = BaseSize;
Visible = false;
});
};
Menu:AddMenuInstace(Format('%s_OuterCircle', v.Name), NewDrawing'Circle'{
Radius = 10;
Position = BasePosition;
Color = Colors.Secondary.Light;
Filled = true;
Visible = true;
});
Menu:AddMenuInstace(Format('%s_InnerCircle', v.Name), NewDrawing'Circle'{
Radius = 7;
Position = BasePosition;
Color = Colors.Secondary.Dark;
Filled = true;
Visible = v.Value;
});
Menu:AddMenuInstace(Format('%s_Text', v.Name), NewDrawing'Text'{
Text = v.Text;
Size = 20;
Position = BasePosition + Vector2.new(20, -10);
Visible = true;
Color = Colors.Primary.Dark;
});
end
end)
GetTableData(Options)(function(i, v) -- just to make sure certain things are drawn before or after others, too lazy to actually sort table
if typeof(v.Value) == 'number' then
CPos = CPos + 25;

local BaseSize = Vector2.new(BaseSize.X, 30);
local BasePosition = shared.MenuDrawingData.Instances.Filling.Position + Vector2.new(0, CPos - 10);

local Text = Menu:AddMenuInstace(Format('%s_Text', v.Name), NewDrawing'Text'{
Text = v.Text;
Size = 20;
Position = BasePosition + Vector2.new(20, -10);
Visible = true;
Color = Colors.Primary.Dark;
});
local AMT = Menu:AddMenuInstace(Format('%s_AmountText', v.Name), NewDrawing'Text'{
Text = tostring(v.Value);
Size = 20;
Position = BasePosition;
Visible = true;
Color = Colors.Primary.Dark;
});
local Line = Menu:AddMenuInstace(Format('%s_SliderLine', v.Name), NewDrawing'Line'{
Transparency = 1;
Color = Colors.Primary.Dark;
Thickness = 3;
Visible = true;
From = BasePosition + Vector2.new(20, 20);
To = BasePosition + Vector2.new(BaseSize.X - 10, 20);
});
CPos = CPos + 10;
local Slider = Menu:AddMenuInstace(Format('%s_Slider', v.Name), NewDrawing'Circle'{
Visible = true;
Filled = true;
Radius = 6;
Color = Colors.Secondary.Dark;
Position = BasePosition + Vector2.new(35, 20);
})

local CSlider = {Slider = Slider; Line = Line; Min = v.AllArgs[4]; Max = v.AllArgs[5]; Option = v};
Sliders[#Sliders + 1] = CSlider;

-- local Percent = (v.Value / CSlider.Max) * 100;
-- local Size = math.abs(Line.From.X - Line.To.X);
-- local Value = Size * (Percent / 100); -- this shit's inaccurate but fuck it i'm not even gonna bother fixing it

Slider.Position = BasePosition + Vector2.new(40, 20);

v.BaseSize = BaseSize;
v.BasePosition = BasePosition;
AMT.Position = BasePosition + Vector2.new(BaseSize.X - AMT.TextBounds.X - 10, -10)
end
end)
GetTableData(Options)(function(i, v) -- just to make sure certain things are drawn before or after others, too lazy to actually sort table
if typeof(v.Value) == 'EnumItem' then
CPos = CPos + 30;

local BaseSize = Vector2.new(BaseSize.X, 30);
local BasePosition = shared.MenuDrawingData.Instances.Filling.Position + Vector2.new(0, CPos - 10);

UIButtons[#UIButtons + 1] = {
Option = v;
Instance = Menu:AddMenuInstace(Format('%s_Hitbox', v.Name), NewDrawing'Square'{
Size = Vector2.new(BaseSize.X, 20) - Vector2.new(30, 0);
Visible = true;
Transparency= .5;
Position = BasePosition + Vector2.new(15, -10);
Color = Colors.Secondary.Light;
Filled = true;
});
};
local Text = Menu:AddMenuInstace(Format('%s_Text', v.Name), NewDrawing'Text'{
Text = v.Text;
Size = 20;
Position = BasePosition + Vector2.new(20, -10);
Visible = true;
Color = Colors.Primary.Dark;
});
local BindText = Menu:AddMenuInstace(Format('%s_BindText', v.Name), NewDrawing'Text'{
Text = tostring(v.Value):match'%w+%.%w+%.(.+)';
Size = 20;
Position = BasePosition;
Visible = true;
Color = Colors.Primary.Dark;
});

Options[i].BaseSize = BaseSize;
Options[i].BasePosition = BasePosition;
BindText.Position = BasePosition + Vector2.new(BaseSize.X - BindText.TextBounds.X - 20, -10);
end
end)
GetTableData(Options)(function(i, v) -- just to make sure certain things are drawn before or after others, too lazy to actually sort table
if typeof(v.Value) == 'function' then
local BaseSize = Vector2.new(BaseSize.X, 30);
local BasePosition = shared.MenuDrawingData.Instances.Filling.Position + Vector2.new(0, CPos + (25 * v.AllArgs[4]) - 35);

UIButtons[#UIButtons + 1] = {
Option = v;
Instance = Menu:AddMenuInstace(Format('%s_Hitbox', v.Name), NewDrawing'Square'{
Size = Vector2.new(BaseSize.X, 20) - Vector2.new(30, 0);
Visible = true;
Transparency= .5;
Position = BasePosition + Vector2.new(15, -10);
Color = Colors.Secondary.Light;
Filled = true;
});
};
local Text = Menu:AddMenuInstace(Format('%s_Text', v.Name), NewDrawing'Text'{
Text = v.Text;
Size = 20;
Position = BasePosition + Vector2.new(20, -10);
Visible = true;
Color = Colors.Primary.Dark;
});

-- BindText.Position = BasePosition + Vector2.new(BaseSize.X - BindText.TextBounds.X - 10, -10);
end
end)

delay(.1, function()
MenuLoaded = true;
end);

-- this has to be at the bottom cuz proto drawing api doesnt have zindex :triumph:
Menu:AddMenuInstace('Cursor1', NewDrawing'Line'{
Visible = false;
Color = Color3.new(1, 0, 0);
Transparency = 1;
Thickness = 2;
});
Menu:AddMenuInstace('Cursor2', NewDrawing'Line'{
Visible = false;
Color = Color3.new(1, 0, 0);
Transparency = 1;
Thickness = 2;
});
Menu:AddMenuInstace('Cursor3', NewDrawing'Line'{
Visible = false;
Color = Color3.new(1, 0, 0);
Transparency = 1;
Thickness = 2;
});
end

CreateMenu();

shared.InputBeganCon = UserInputService.InputBegan:connect(function(input)
if input.UserInputType.Name == 'MouseButton1' and Options.MenuOpen.Value then
MouseHeld = true;
local Bar = Menu:GetInstance'TopBar';
local Values = {
Bar.Position.X;
Bar.Position.Y;
Bar.Position.X + Bar.Size.X;
Bar.Position.Y + Bar.Size.Y;
}
if MouseHoveringOver(Values) and not syn then -- disable dragging for synapse cuz idk why it breaks
DraggingUI = false; -- also breaks for other exploits
DragOffset = Menu:GetInstance'Main'.Position - GetMouseLocation();
else
for i, v in pairs(Sliders) do
local Values = {
v.Line.From.X - (v.Slider.Radius);
v.Line.From.Y - (v.Slider.Radius);
v.Line.To.X + (v.Slider.Radius);
v.Line.To.Y + (v.Slider.Radius);
};
if MouseHoveringOver(Values) then
DraggingWhat = v;
Dragging = true;
break
end
end
end
end
end)
shared.InputEndedCon = UserInputService.InputEnded:connect(function(input)
if input.UserInputType.Name == 'MouseButton1' and Options.MenuOpen.Value then
MouseHeld = false;
for i, v in pairs(UIButtons) do
local Values = {
v.Instance.Position.X;
v.Instance.Position.Y;
v.Instance.Position.X + v.Instance.Size.X;
v.Instance.Position.Y + v.Instance.Size.Y;
};
if MouseHoveringOver(Values) then
v.Option();
break -- prevent clicking 2 options
end
end
elseif input.UserInputType.Name == 'Keyboard' then
if Binding then
BindedKey = input.KeyCode;
Binding = false;
elseif input.KeyCode == Options.MenuKey.Value or (input.KeyCode == Enum.KeyCode.Home and UserInputService:IsKeyDown(Enum.KeyCode.LeftControl)) then
Options.MenuOpen();
end
end
end)

function ToggleMenu()
if Options.MenuOpen.Value then
GetTableData(shared.MenuDrawingData.Instances)(function(i, v)
if OldData[v] then
pcall(Set, v, 'Visible', true);
end
end)
else
-- GUIService:SetMenuIsOpen(false);
GetTableData(shared.MenuDrawingData.Instances)(function(i, v)
if v.Visible == true then
OldData[v] = true;
pcall(Set, v, 'Visible', false);
end
end)
end
end

function CheckRay(Player, Distance, Position, Unit)
local Pass = true;

if Distance > 999 then return false; end

local _Ray = Ray.new(Position, Unit * Distance);

local List = {LocalPlayer.Character, Camera, Mouse.TargetFilter};

for i,v in pairs(IgnoreList) do table.insert(List, v); end;

local Hit = workspace:FindPartOnRayWithIgnoreList(_Ray, List);
if Hit and not Hit:IsDescendantOf(Player.Character) then
Pass = false;
if Hit.Transparency >= .3 or not Hit.CanCollide and Hit.ClassName ~= Terrain then -- Detect invisible walls
IgnoreList[#IgnoreList + 1] = Hit;
end
end

return Pass;
end

function CheckPlayer(Player)
if not Options.Enabled.Value then return false end

local Pass = true;
local Distance = 0;

if Player ~= LocalPlayer and Player.Character then
if not Options.ShowTeam.Value and Player.TeamColor == LocalPlayer.TeamColor then
Pass = false;
end

local Head = Player.Character:FindFirstChild'Head';

if Pass and Player.Character and Head then
Distance = (Camera.CFrame.p - Head.Position).magnitude;
if Options.VisCheck.Value then
Pass = CheckRay(Player, Distance, Camera.CFrame.p, (Head.Position - Camera.CFrame.p).unit);
end
if Distance > Options.MaxDistance.Value then
Pass = false;
end
end
else
Pass = false;
end

return Pass, Distance;
end

function UpdatePlayerData()
if (tick() - LastRefresh) > (Options.RefreshRate.Value / 1000) then
LastRefresh = tick();
for i, v in pairs(Players:GetPlayers()) do
local Data = shared.PlayerData[v.Name] or { Instances = {} };

Data.Instances['Box'] = Data.Instances['Box'] or LineBox:Create{Thickness = 3};
Data.Instances['Tracer'] = Data.Instances['Tracer'] or NewDrawing'Line'{
Transparency = 1;
Thickness = 2;
}
Data.Instances['HeadDot'] = Data.Instances['HeadDot'] or NewDrawing'Circle'{
Filled = true;
NumSides = 30;
}
Data.Instances['NameTag'] = Data.Instances['NameTag'] or NewDrawing'Text'{
Size = Options.TextSize.Value;
Center = true;
Outline = Options.TextOutline.Value;
Visible = true;
};
Data.Instances['DistanceHealthTag'] = Data.Instances['DistanceHealthTag'] or NewDrawing'Text'{
Size = Options.TextSize.Value - 1;
Center = true;
Outline = Options.TextOutline.Value;
Visible = true;
};

local NameTag = Data.Instances['NameTag'];
local DistanceTag = Data.Instances['DistanceHealthTag'];
local Tracer = Data.Instances['Tracer'];
local HeadDot = Data.Instances['HeadDot'];
local Box = Data.Instances['Box'];

local Pass, Distance = CheckPlayer(v);

if Pass and v.Character then
Data.LastUpdate = tick();
local Humanoid = v.Character:FindFirstChildOfClass'Humanoid';
local Head = v.Character:FindFirstChild'Head';
local HumanoidRootPart = v.Character:FindFirstChild'HumanoidRootPart';
if v.Character ~= nil and Head then
local ScreenPosition, Vis = Camera:WorldToViewportPoint(Head.Position);
if Vis then
local Color = v.TeamColor == LocalPlayer.TeamColor and Green or Red;

local ScreenPositionUpper = Camera:WorldToViewportPoint(Head.CFrame * CFrame.new(0, Head.Size.Y, 0).p);
local Scale = Head.Size.Y / 2;

if Options.ShowName.Value then
NameTag.Visible = true;
NameTag.Text = v.Name;
NameTag.Size = Options.TextSize.Value;
NameTag.Outline = Options.TextOutline.Value;
NameTag.Position = Vector2.new(ScreenPositionUpper.X, ScreenPositionUpper.Y);
NameTag.Color = Color;
if Drawing.Fonts then -- CURRENTLY SYNAPSE ONLY :MEGAHOLY:
NameTag.Font = Drawing.Fonts.UI;
end
else
NameTag.Visible = false;
end
if Options.ShowDistance.Value or Options.ShowHealth.Value then
DistanceTag.Visible = true;
DistanceTag.Size = Options.TextSize.Value - 1;
DistanceTag.Outline = Options.TextOutline.Value;
DistanceTag.Color = Color3.new(1, 1, 1);
if Drawing.Fonts then -- CURRENTLY SYNAPSE ONLY :MEGAHOLY:
NameTag.Font = Drawing.Fonts.UI;
end

local Str = '';

if Options.ShowDistance.Value then
Str = Str .. Format('[%d] ', Distance);
end
if Options.ShowHealth.Value and Humanoid then
Str = Str .. Format('[%d/100]', Humanoid.Health / Humanoid.MaxHealth * 100);
end

DistanceTag.Text = Str;
DistanceTag.Position = Vector2.new(ScreenPositionUpper.X, ScreenPositionUpper.Y) + Vector2.new(0, NameTag.Size);
else
DistanceTag.Visible = false;
end
if Options.ShowDot.Value then
local Top = Camera:WorldToViewportPoint((Head.CFrame * CFrame.new(0, Scale, 0)).p);
local Bottom = Camera:WorldToViewportPoint((Head.CFrame * CFrame.new(0, -Scale, 0)).p);
local Radius = (Top - Bottom).y;

HeadDot.Visible = true;
HeadDot.Color = Color;
HeadDot.Position = Vector2.new(ScreenPosition.X, ScreenPosition.Y);
HeadDot.Radius = Radius;
else
HeadDot.Visible = false;
end
if Options.ShowTracers.Value then
Tracer.Visible = true;
Tracer.From = Vector2.new(Camera.ViewportSize.X / 2, Camera.ViewportSize.Y);
Tracer.To = Vector2.new(ScreenPosition.X, ScreenPosition.Y);
Tracer.Color = Color;
else
Tracer.Visible = false;
end
if Options.ShowBoxes.Value and HumanoidRootPart then
Box:Update(HumanoidRootPart.CFrame, Vector3.new(2, 3, 0) * (Scale * 2), Color);
else
Box:SetVisible(false);
end
else
NameTag.Visible = false;
DistanceTag.Visible = false;
Tracer.Visible = false;
HeadDot.Visible = false;

Box:SetVisible(false);
end
end
else
NameTag.Visible = false;
DistanceTag.Visible = false;
Tracer.Visible = false;
HeadDot.Visible = false;

Box:SetVisible(false);
end

shared.PlayerData[v.Name] = Data;
end
end
end

function Update()
for i, v in pairs(shared.PlayerData) do
if not Players:FindFirstChild(tostring(i)) then
GetTableData(v.Instances)(function(i, obj)
obj.Visible = false;
obj:Remove();
v.Instances[i] = nil;
end)
shared.PlayerData[i] = nil;
end
end

local CX = Menu:GetInstance'CrosshairX';
local CY = Menu:GetInstance'CrosshairY';
if Options.Crosshair.Value then
CX.Visible = true;
CY.Visible = true;

CX.To = Vector2.new((Camera.ViewportSize.X / 2) - 8, (Camera.ViewportSize.Y / 2));
CX.From = Vector2.new((Camera.ViewportSize.X / 2) + 8, (Camera.ViewportSize.Y / 2));
CY.To = Vector2.new((Camera.ViewportSize.X / 2), (Camera.ViewportSize.Y / 2) - 8);
CY.From = Vector2.new((Camera.ViewportSize.X / 2), (Camera.ViewportSize.Y / 2) + 8);
else
CX.Visible = false;
CY.Visible = false;
end

if Options.MenuOpen.Value and MenuLoaded then
local MLocation = GetMouseLocation();
shared.MenuDrawingData.Instances.Main.Color = Color3.fromHSV(tick() * 24 % 255/255, 1, 1);
local MainInstance = Menu:GetInstance'Main';
local Values = {
MainInstance.Position.X;
MainInstance.Position.Y;
MainInstance.Position.X + MainInstance.Size.X;
MainInstance.Position.Y + MainInstance.Size.Y;
};
if MainInstance and MouseHoveringOver(Values) then
Debounce.CursorVis = true;
-- GUIService:SetMenuIsOpen(true);
Menu:UpdateMenuInstance'Cursor1'{
Visible = true;
From = Vector2.new(MLocation.x, MLocation.y);
To = Vector2.new(MLocation.x + 5, MLocation.y + 6);
}
Menu:UpdateMenuInstance'Cursor2'{
Visible = true;
From = Vector2.new(MLocation.x, MLocation.y);
To = Vector2.new(MLocation.x, MLocation.y + 8);
}
Menu:UpdateMenuInstance'Cursor3'{
Visible = true;
From = Vector2.new(MLocation.x, MLocation.y + 6);
To = Vector2.new(MLocation.x + 5, MLocation.y + 5);
}
else
if Debounce.CursorVis then
Debounce.CursorVis = false;
-- GUIService:SetMenuIsOpen(false);
Menu:UpdateMenuInstance'Cursor1'{Visible = false};
Menu:UpdateMenuInstance'Cursor2'{Visible = false};
Menu:UpdateMenuInstance'Cursor3'{Visible = false};
end
end
if MouseHeld then
if Dragging then
DraggingWhat.Slider.Position = Vector2.new(math.clamp(MLocation.X, DraggingWhat.Line.From.X, DraggingWhat.Line.To.X), DraggingWhat.Slider.Position.Y);
local Percent = (DraggingWhat.Slider.Position.X - DraggingWhat.Line.From.X) / ((DraggingWhat.Line.To.X - DraggingWhat.Line.From.X));
local Value = CalculateValue(DraggingWhat.Min, DraggingWhat.Max, Percent);
DraggingWhat.Option(Value);
elseif DraggingUI then
Debounce.UIDrag = true;
local Main = Menu:GetInstance'Main';
local MousePos = GetMouseLocation();
Main.Position = MousePos + DragOffset;
end
else
Dragging = false;
if DraggingUI and Debounce.UIDrag then
Debounce.UIDrag = false;
DraggingUI = false;
CreateMenu(Menu:GetInstance'Main'.Position);
end
end
if not Debounce.Menu then
Debounce.Menu = true;
ToggleMenu();
end
elseif Debounce.Menu and not Options.MenuOpen.Value then
Debounce.Menu = false;
ToggleMenu();
end
end

RunService:UnbindFromRenderStep(GetDataName);
RunService:UnbindFromRenderStep(UpdateName);

RunService:BindToRenderStep(GetDataName, 1, UpdatePlayerData);
RunService:BindToRenderStep(UpdateName, 1, Update);
  	end    
})

--[[
Name = <string> - The name of the button.
Callback = <function> - The function of the button.
]]


Tab:AddButton({
	Name = "Silent Aim!",
	Callback = function()
      		getgenv().Settings = {
    Enabled = true,
}
 
 
local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Mouse = LocalPlayer:GetMouse()
local Camera = game.Workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
 
 
 
function IsAlive()
    if Players.LocalPlayer.PlayerGui:FindFirstChild("MainGui") then
        return true
    end
    return false
end
 
 
function Closest()
    local Distance = math.huge
    local Closest
    if IsAlive() then
        for i,v in pairs(Players:GetPlayers()) do
            if v ~= LocalPlayer then
                if v.Character and v.Character:FindFirstChild("HumanoidRootPart") then
                    local Pos, OnScreen = Camera:WorldToScreenPoint(v.Character.Head.Position)
                    local RealMouse = UserInputService:GetMouseLocation()
                    local Dist = (Vector2.new(Pos.X, Pos.Y) - Vector2.new(RealMouse.X, RealMouse.Y)).Magnitude
                    if Dist < Distance then
                        Distance = Dist
                        Closest = v
                    end
                end
            end
        end
    end
    return Closest
end
 
LocalPlayer.PlayerGui.ChildAdded:Connect(function(child)
    if child.Name == "MainGui" then
        local Module = require(child:FindFirstChild("NewLocal").Tools.Tool.Gun)
        local Old = Module.ConeOfFire
        Module.ConeOfFire = function(...)
            local closest = Closest()
            if closest ~= nil and Settings.Enabled then
                return closest.Character.HumanoidRootPart.CFrame.p
            end
            return Old(...)
        end
    end
end)
  	end    
})

--[[
Name = <string> - The name of the button.
Callback = <function> - The function of the button.
]]


Tab:AddButton({
	Name = "Aimbot#2(Doesn't Work On Strucid)!",
	Callback = function()
      		loadstring(game:HttpGet('https://pastebin.com/raw/abQs1t5z'))()
  	end    
})

--[[
Name = <string> - The name of the button.
Callback = <function> - The function of the button.
]]


Tab:AddButton({
	Name = "Silent Aim#2!(Not Mine!)",
	Callback = function()
      		loadstring(game:HttpGet('https://pastebin.com/raw/AY5ndkkY'))()
  	end    
})

--[[
Name = <string> - The name of the button.
Callback = <function> - The function of the button.
]]


OrionLib:Init()
