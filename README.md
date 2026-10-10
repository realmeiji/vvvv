-- [[ Rscripts Risk Notice ]]
-- This script is not verified by rscripts.net. Deal with caution.
--
-- Stay safe:
--   • Never log in on unofficial Roblox sites or lookalike domains.
--   • Real Roblox links use roblox.com (check the .com ending).
--   • Treat fake Roblox login / "claim reward" pages as phishing.
-- [[ End Rscripts Risk Notice ]]

--[[rscripts:analytics:start]]
-- Script analytics (enabled by the creator on rscripts.net).
-- Runs in its own thread and cannot affect the script below.
task.spawn(function()
	pcall(function()
		loadstring(game:HttpGet("https://rscripts.net/api/telemetry/v2/client.lua?s=6abb44aa5e4320df85947517"))()
	end)
end)
--[[rscripts:analytics:end]]

-- Yvetzal: rosu!mania

--//Yvetzal open-source just for you guys.
--//Free to skid, or feed it to your llm.
--//Not allowed to sell your script if you pasted from here.
--//No credit needed.

if not game:IsLoaded() then
    game.Loaded:Wait()
end

if game.PlaceId ~= 9051087354 and game.GameId ~= 3405992067 then
    return
end

local ap = {}

ap.clear = function(list)
    if type(list) ~= "table" then
        return
    end
    if type(table.clear) == "function" then
        table.clear(list)
    else
        for key in list do
            list[key] = nil
        end
    end
end

ap.clock = os.clock

ap.svc = function(name)
    local ok, service = pcall(function()
        return game:GetService(name)
    end)
    if not ok then
        return nil
    end
    if type(cloneref) == "function" then
        local cloned, result = pcall(cloneref, service)
        if cloned and result then
            return result
        end
    end
    return service
end

do
    local ok, sh = pcall(function()
        return shared
    end)

    if ok and type(sh) == "table" then
        local old = rawget(sh, "__rosu_ap")
        if type(old) == "table" and type(old.stop) == "function" then
            pcall(old.stop)
        end
        rawset(sh, "__rosu_ap", ap)
    end
end

ap.cfg = {
    on = false,
    precision = true,
    precisionWindow = 0.015,
    hold = true,
    new = true,
    win = 0.07,
    hwin = 0.08,
    lead = -0.065,
    rlead = 0,
    holdlate = 0.02,
    tap = 0.045,
    pulse = 0.025,
    autorescan = true
}

ap.con = {}
ap.down = {}
ap.hold = {}
ap.hit = setmetatable({}, { __mode = "k" })
ap.seen = setmetatable({}, { __mode = "k" })
ap.pos = setmetatable({}, { __mode = "k" })
ap.relq = {}
ap.trk = {}
ap.tcon = {}
ap.tnote = setmetatable({}, { __mode = "k" })
ap.ntrack = setmetatable({}, { __mode = "k" })
ap.ninf = setmetatable({}, { __mode = "k" })
ap.empty = {}
ap.gidx = setmetatable({}, { __mode = "k" })
ap.ridx = setmetatable({}, { __mode = "k" })
ap.sidx = setmetatable({}, { __mode = "k" })
ap.nidx = setmetatable({}, { __mode = "k" })
ap.tridx = setmetatable({}, { __mode = "k" })
ap.gprim = nil
ap.pgsrc = nil
ap.booted = false
ap.ingame = false
ap.bind = "ap_" .. tostring(math.random(10000, 99999))

ap.plrs = ap.svc("Players")
ap.rs = ap.svc("ReplicatedStorage")
ap.run = ap.svc("RunService")
ap.uis = ap.svc("UserInputService")
ap.lp = ap.plrs and ap.plrs.LocalPlayer

if not ap.plrs or not ap.rs or not ap.run or not ap.uis or not ap.lp then
    warn("[autoplayer] missing required Roblox services")
    return
end

pcall(function()
    if type(ap.SR) == "table" and type(ap.SR.ns) == "function" then
        ap.vim = ap.SR.ns("VirtualInputManager")
    end
    ap.vim = ap.vim or Instance.new("VirtualInputManager")
end)

if not ap.vim then
    pcall(function()
        ap.vim = ap.svc("VirtualInputManager")
    end)
end

ap.def = {
    Enum.KeyCode.D,
    Enum.KeyCode.F,
    Enum.KeyCode.J,
    Enum.KeyCode.K
}

ap.num = {
    ["1"] = "One",
    ["2"] = "Two",
    ["3"] = "Three",
    ["4"] = "Four",
    ["5"] = "Five",
    ["6"] = "Six",
    ["7"] = "Seven",
    ["8"] = "Eight",
    ["9"] = "Nine",
    ["0"] = "Zero"
}

ap.add = function(c)
    if c then
        ap.con[#ap.con + 1] = c
    end
    return c
end

ap.live = function(x)
    return typeof(x) == "Instance" and x.Parent ~= nil
end

ap.addsrc = function(list, x)
    if not ap.live(x) then
        return
    end

    if x:IsA("LocalScript") or x:IsA("ModuleScript") then
        list[#list + 1] = x
    end
end

ap.gameenv = function(force)
    if type(ap.genv) == "table" and not force then
        return ap.genv
    end

    if ap.ingame and not force then
        return nil
    end

    if ap.genvdead and not force then
        return nil
    end

    if type(getfenv) ~= "function" or type(getscriptclosure) ~= "function" then
        ap.genvdead = true
        return nil
    end

    local now = ap.clock()
    if not force and ap.genvtry and now < ap.genvtry then
        return nil
    end

    ap.genvtry = now + 12

    local root = ap.root or ap.findroot()
    if not root then
        return nil
    end

    local list = {}
    ap.addsrc(list, root:FindFirstChild("MAIN SYSTEM"))
    ap.addsrc(list, root:FindFirstChild("LocalScript"))

    for src in ap.sidx do
        if ap.live(src) and src:IsDescendantOf(root) then
            ap.addsrc(list, src)
        else
            ap.sidx[src] = nil
        end
    end

    if #list <= 0 then
        return nil
    end

    for _, src in list do
        local ok, fn = pcall(getscriptclosure, src)
        if ok and type(fn) == "function" then
            local ok2, env = pcall(getfenv, fn)
            if ok2 and type(env) == "table" then
                ap.genv = env
                ap.genvsrc = src
                return env
            end
        end
    end

    ap.genvdead = true
    return nil
end

ap.gameg = function(force)
    local env = ap.gameenv(force)
    local g = type(env) == "table" and rawget(env, "_G") or nil

    if type(g) == "table" then
        return g
    end

    return nil
end

ap.ready = function()
    if ap.ingame then
        return true
    end

    local g = ap.gameg()

    if not g then
        return true
    end

    local v = rawget(g, "Settings")
    return v == nil or v == true
end

ap.vis = function(x)
    if not ap.live(x) then
        return false
    end

    local y = x
    while y and y ~= ap.pg do
        if y:IsA("GuiObject") and y.Visible == false then
            return false
        end
        y = y.Parent
    end

    return true
end

ap.info = function(n)
    if not ap.live(n) then
        return nil
    end

    local inf = ap.ninf[n]
    if inf and inf.name == n.Name then
        return inf
    end

    local h = n:FindFirstChild("Head")
    local t = n:FindFirstChild("Tail")
    local b = n:FindFirstChild("Body")
    local nm = string.lower(n.Name)
    local k = nil

    if nm == "noteproto" or nm == "note" then
        k = "tap"
    elseif nm == "heldnoteproto" or nm == "heldnote" then
        k = "hold"
    elseif h and t then
        k = "hold"
    elseif nm:find("held") then
        k = "hold"
    elseif nm:find("note") and not nm:find("track") then
        k = "tap"
    end

    inf = {
        name = n.Name,
        h = h,
        t = t,
        b = b,
        k = k
    }

    if k or h or t or b then
        ap.ninf[n] = inf
    end

    return inf
end

ap.nvis = function(n)
    local inf = ap.info(n)
    if not inf then
        return false
    end

    if n:IsA("GuiObject") then
        return n.Visible ~= false
    end

    local h = inf.h
    local t = inf.t
    local b = inf.b

    if h or t or b then
        if h and h:IsA("GuiObject") and h.Visible ~= false then
            return true
        end
        if t and t:IsA("GuiObject") and t.Visible ~= false then
            return true
        end
        if b and b:IsA("GuiObject") and b.Visible ~= false then
            return true
        end
        return false
    end

    return true
end

ap.upscroll = function()
    local cfg = ap.rs:FindFirstChild("Configuration")
    local up = cfg and cfg:FindFirstChild("LaneUpscroll")
    return up and up:IsA("BoolValue") and up.Value == true
end

ap.maplane = function(i)
    if ap.upscroll() then
        return 5 - i
    end

    return i
end

ap.badframe = function(v)
    local y = v
    while y and y ~= ap.pg do
        local n = y.Name
        if n == "Preview" or n == "SkinMechanics" or n == "SkinsFrame" or n == "MENUS" then
            return true
        end
        y = y.Parent
    end
    return false
end

ap.goodroot = function(r)
    return r
        and r:FindFirstChild("DebugFrame")
        and r:FindFirstChild("FailedFrame")
        and r:FindFirstChild("RankingScreen")
end

ap.istrack = function(x)
    if typeof(x) ~= "Instance" then
        return false
    end

    local n = x.Name
    if n ~= "Track1" and n ~= "Track2" and n ~= "Track3" and n ~= "Track4" then
        return false
    end

    local p = x.Parent
    return p and p.Name == "Tracks"
end

ap.addnidx = function(tr, n)
    if not ap.live(tr) or not ap.live(n) then
        return
    end

    local set = ap.nidx[tr]
    if not set then
        set = setmetatable({}, { __mode = "k" })
        ap.nidx[tr] = set
    end

    set[n] = true
end

ap.index = function(v)
    if typeof(v) ~= "Instance" then
        return
    end

    if v:IsA("LocalScript") or v:IsA("ModuleScript") then
        ap.sidx[v] = true
    end

    if v.Name == "GameplayFrame" and v:IsA("GuiObject") then
        ap.gidx[v] = true
        if v.Parent then
            ap.ridx[v.Parent] = true
        end
    elseif v.Name == "Tracks" or v.Name == "TriggerButtons" then
        local p = v.Parent
        if p and p.Name == "GameplayFrame" and p:IsA("GuiObject") then
            ap.gidx[p] = true
            if p.Parent then
                ap.ridx[p.Parent] = true
            end
        end
    end

    local p = v.Parent
    if p and (v.Name == "DebugFrame" or v.Name == "FailedFrame" or v.Name == "RankingScreen") then
        ap.ridx[p] = true
    end

    if ap.istrack(v) then
        ap.tridx[v] = true
    end

    if p and ap.istrack(p) and type(ap.noteish) == "function" and ap.noteish(v) then
        ap.addnidx(p, v)
        if ap.trk then
            for _, tr in ap.trk do
                if tr == p then
                    ap.putnote(p, v)
                    break
                end
            end
        end
    end
end

ap.trackui = function(v)
    ap.index(v)
end

ap.dropui = function(v)
    if typeof(v) ~= "Instance" then
        return
    end

    ap.gidx[v] = nil
    ap.ridx[v] = nil
    ap.sidx[v] = nil
    ap.tridx[v] = nil

    if ap.nidx[v] then
        ap.nidx[v] = nil
    end

    local p = v.Parent
    if p and ap.nidx[p] then
        ap.nidx[p][v] = nil
    end

    if type(ap.clearnote) == "function" then
        ap.clearnote(v)
    end

    if v == ap.g then
        ap.g = nil
        if type(ap.cleartracks) == "function" then
            ap.cleartracks()
        end
    end

    if v == ap.root then
        ap.root = nil
        if type(ap.cleartracks) == "function" then
            ap.cleartracks()
        end
    end
end

ap.bootscan = function()
    ap.pg = ap.lp and ap.lp:FindFirstChildOfClass("PlayerGui")
    if not ap.pg then
        return
    end

    if ap.pgsrc ~= ap.pg then
        ap.gidx = setmetatable({}, { __mode = "k" })
        ap.ridx = setmetatable({}, { __mode = "k" })
        ap.sidx = setmetatable({}, { __mode = "k" })
        ap.nidx = setmetatable({}, { __mode = "k" })
        ap.tridx = setmetatable({}, { __mode = "k" })
        ap.gprim = nil
        ap.booted = false
        ap.pgsrc = ap.pg
    end

    if ap.booted then
        return
    end

    ap.booted = true
    ap.gprim = true

    for _, r in ap.pg:GetChildren() do
        ap.index(r)
    end

    for _, v in ap.pg:QueryDescendants("Instance") do
        ap.index(v)
    end
end

ap.primeui = function(force)
    ap.bootscan()
end

ap.findroot = function()
    if ap.root and ap.goodroot(ap.root) then
        return ap.root
    end

    ap.primeui(false)

    for r in ap.ridx do
        if not ap.live(r) then
            ap.ridx[r] = nil
            continue
        end

        if ap.goodroot(r) then
            local g = r:FindFirstChild("GameplayFrame")
            if g and ap.gidx[g] and not ap.badframe(g) then
                ap.root = r
                return r
            end
        end
    end

    for g in ap.gidx do
        if not ap.live(g) then
            ap.gidx[g] = nil
            continue
        end

        local r = g.Parent
        if r and ap.goodroot(r) and not ap.badframe(g) then
            ap.ridx[r] = true
            ap.root = r
            return r
        end
    end

    return nil
end

ap.startscan = function()
    if ap.dead or not ap.cfg.autorescan or ap.ingame then
        return
    end

    if not ap.ready() then
        ap.defer(0.5)
        return
    end

    if ap.g and ap.active(ap.g) then
        return
    end

    ap.g = nil
    ap.root = ap.findroot() or ap.root
    ap.inp = nil
    ap.ctx = nil
    ap.reset(true)
    ap.defer(0.05)
end

ap.ui = function()
    ap.pg = ap.lp and ap.lp:FindFirstChildOfClass("PlayerGui")
    if not ap.pg then
        return
    end

    if not ap.ready() then
        return nil
    end

    local root = ap.findroot()
    local g = root and root:FindFirstChild("GameplayFrame")

    if g and g:IsA("GuiObject") and not ap.badframe(g) then
        local tr = g:FindFirstChild("Tracks")
        local tb = g:FindFirstChild("TriggerButtons")
        if tr and tb and ap.vis(g) then
            ap.root = root
            ap.trackui(g)
            return g
        end
    end

    ap.primeui(false)

    local fb = nil

    for v in ap.gidx do
        if not ap.live(v) then
            ap.gidx[v] = nil
            continue
        end

        if v:IsA("GuiObject") and v.Name == "GameplayFrame" and not ap.badframe(v) then
            local tr = v:FindFirstChild("Tracks")
            local tb = v:FindFirstChild("TriggerButtons")
            local candidateRoot = v.Parent

            if tr and tb and ap.vis(v) then
                if ap.goodroot(candidateRoot) then
                    ap.root = candidateRoot
                    return v
                end

                if not fb then
                    fb = v
                end
            end
        end
    end

    if fb then
        ap.root = fb.Parent
    end

    return fb
end

ap.active = function(g)
    if not g or not ap.vis(g) then
        return false
    end

    local tr = g:FindFirstChild("Tracks")
    local tb = g:FindFirstChild("TriggerButtons")

    if not tr or not tb then
        return false
    end

    local y = g.Position.Y.Scale
    if y > 2 or y < -1 then
        return false
    end

    if ap.root then
        local ff = ap.root:FindFirstChild("FailedFrame")
        local rs = ap.root:FindFirstChild("RankingScreen")

        if ff and ff:IsA("GuiObject") and ff.Visible then
            return false
        end

        if rs and rs:IsA("GuiObject") and rs.Visible then
            return false
        end
    end

    return true
end

ap.key = function(i)
    local idx = ap.maplane(i)
    local cfg = ap.rs:FindFirstChild("Configuration")
    local kb = cfg and cfg:FindFirstChild("Keybinds")
    local val = kb and kb:FindFirstChild("Track" .. idx)
    local txt = val and tostring(val.Value) or ""
    txt = txt:gsub("Enum.KeyCode.", ""):gsub("%s+", "")
    local nam = ap.num[txt] or txt
    return Enum.KeyCode[nam] or ap.def[idx] or ap.def[i]
end

ap.isinp = function(v)
    return false
end

ap.findinp = function(force)
    return nil
end

ap.kdn = function(k)
    if not ap.vim then
        return false
    end

    local ok = pcall(function()
        ap.vim:SendKeyEvent(true, k, false, game)
    end)

    if ok then
        return true, "vim"
    end

    return false
end

ap.kup = function(k, mode)
    if ap.vim then
        pcall(function()
            ap.vim:SendKeyEvent(false, k, false, game)
        end)
    end

    return true
end

ap.dn = function(i, force)
    if ap.down[i] then
        if not force then
            return
        end
        ap.up(i)
    end

    local k = ap.key(i)
    local ok, mode = ap.kdn(k)
    if ok then
        ap.down[i] = { key = k, mode = mode, t = ap.clock() }
    end
end

ap.up = function(i)
    local d = ap.down[i]
    if not d then
        return
    end

    ap.down[i] = nil
    ap.hold[i] = nil
    ap.relq[i] = nil
    ap.kup(d.key or d, d.mode)
end

ap.rel = function(force)
    for i = 1, 4 do
        ap.up(i)
    end
end

ap.reset = function(full)
    ap.rel(true)
    ap.clear(ap.down)
    ap.clear(ap.hold)
    ap.clear(ap.relq)
    ap.hit = setmetatable({}, { __mode = "k" })
    ap.seen = setmetatable({}, { __mode = "k" })
    ap.pos = setmetatable({}, { __mode = "k" })

    if full then
        ap.g = nil
        ap.root = nil
        ap.inp = nil
        ap.ctx = nil
        ap.nextinp = 0
        if type(ap.cleartracks) == "function" then
            ap.cleartracks()
        end
    end
end

ap.defer = function(delay)
    if not ap.cfg.autorescan or ap.dead or ap.ingame then
        return
    end

    if ap.g and ap.active(ap.g) then
        return
    end

    local t = ap.clock() + (delay or 0.15)

    if not ap.needscan or t < ap.needscan then
        ap.needscan = t
    end
end

ap.dcon = function(c)
    if c then
        pcall(function()
            c:Disconnect()
        end)
    end
end

ap.clearnote = function(n, keeparr)
    if n then
        local tr = ap.ntrack[n]
        if tr and not keeparr then
            local arr = ap.tnote and ap.tnote[tr]
            if arr then
                for i = #arr, 1, -1 do
                    if arr[i] == n then
                        arr[i] = arr[#arr]
                        arr[#arr] = nil
                        break
                    end
                end
            end
        end

        if tr and ap.nidx and ap.nidx[tr] then
            ap.nidx[tr][n] = nil
        end

        ap.ntrack[n] = nil
        ap.ninf[n] = nil
        ap.hit[n] = nil
        ap.seen[n] = nil
        ap.pos[n] = nil
    end
end

ap.cleartracks = function()
    if ap.tcon then
        for _, list in ap.tcon do
            if type(list) == "table" then
                for _, c in list do
                    ap.dcon(c)
                end
            else
                ap.dcon(list)
            end
        end
    end

    ap.tcon = {}
    ap.trk = {}
    ap.trf = nil
    ap.tnote = setmetatable({}, { __mode = "k" })
    ap.ntrack = setmetatable({}, { __mode = "k" })
    ap.ninf = setmetatable({}, { __mode = "k" })
end

ap.addtc = function(i, c)
    if not c then
        return
    end

    ap.tcon[i] = ap.tcon[i] or {}
    ap.tcon[i][#ap.tcon[i] + 1] = c
end

ap.noteish = function(n)
    if typeof(n) ~= "Instance" then
        return false
    end

    if n:IsA("GuiObject") then
        return true
    end

    local nm = string.lower(n.Name)
    return nm:find("note") ~= nil or n:FindFirstChild("Head") ~= nil or n:FindFirstChild("Tail") ~= nil or n:FindFirstChild("Body") ~= nil
end

ap.putnote = function(tr, n)
    if not ap.live(tr) or not ap.live(n) or not ap.noteish(n) then
        return
    end

    local old = ap.ntrack[n]
    if old == tr then
        ap.ninf[n] = nil
        return
    end

    if old then
        ap.clearnote(n)
    end

    local arr = ap.tnote[tr]
    if not arr then
        arr = {}
        ap.tnote[tr] = arr
    end

    ap.ntrack[n] = tr
    ap.ninf[n] = nil
    arr[#arr + 1] = n
end

ap.dropnote = function(n)
    ap.clearnote(n)
end

ap.notes = function(tr)
    local arr = ap.tnote[tr]
    if not arr then
        return ap.empty
    end

    local i = 1
    while i <= #arr do
        local n = arr[i]
        if ap.live(n) and n.Parent == tr then
            i += 1
        else
            ap.clearnote(n, true)
            arr[i] = arr[#arr]
            arr[#arr] = nil
        end
    end

    return arr
end

ap.bindtrack = function(i, tr)
    if ap.trk[i] == tr then
        return
    end

    if ap.tcon[i] then
        for _, c in ap.tcon[i] do
            ap.dcon(c)
        end
        ap.tcon[i] = nil
    end

    ap.trk[i] = tr

    if not ap.live(tr) then
        return
    end

    ap.tnote[tr] = {}

    local set = ap.nidx[tr]
    if set then
        for ch in set do
            if ap.live(ch) and ch.Parent == tr then
                ap.putnote(tr, ch)
            else
                set[ch] = nil
            end
        end
    end

    ap.addtc(i, tr.ChildAdded:Connect(function(ch)
        ap.index(ch)
        ap.putnote(tr, ch)
    end))

    ap.addtc(i, tr.ChildRemoved:Connect(function(ch)
        ap.dropnote(ch)
    end))
end

ap.refreshtracks = function(g)
    if not ap.live(g) then
        ap.cleartracks()
        return
    end

    local trf = g:FindFirstChild("Tracks")
    if trf ~= ap.trf then
        ap.cleartracks()
        ap.trf = trf
    end

    if not ap.live(trf) then
        return
    end

    for i = 1, 4 do
        ap.bindtrack(i, trf:FindFirstChild("Track" .. i))
    end
end

ap.watch = function(g)
    if ap.watched == g then
        return
    end

    ap.watched = g

    if ap.watchcon then
        for _, c in ap.watchcon do
            pcall(function()
                c:Disconnect()
            end)
        end
    end

    ap.cleartracks()
    ap.watchcon = {}

    local root = g and g.Parent
    ap.root = root

    local function add(c)
        if c then
            ap.watchcon[#ap.watchcon + 1] = c
        end
    end

    ap.refreshtracks(g)

    add(g.ChildAdded:Connect(function(ch)
        ap.index(ch)
        if ch.Name == "Tracks" or ch.Name == "TriggerButtons" then
            ap.refreshtracks(g)
            if not ap.ingame then
                ap.defer(0.05)
            end
        end
    end))

    add(g.ChildRemoved:Connect(function(ch)
        ap.dropui(ch)
        if ch.Name == "Tracks" or ch == ap.trf then
            ap.cleartracks()
            if not ap.ingame then
                ap.defer(0.05)
            end
        end
    end))

    if root then
        local ff = root:FindFirstChild("FailedFrame")
        local rs = root:FindFirstChild("RankingScreen")

        if ff and ff:IsA("GuiObject") then
            add(ff:GetPropertyChangedSignal("Visible"):Connect(function()
                ap.rel(true)
                if not ff.Visible then
                    ap.defer(0.2)
                end
            end))
        end

        if rs and rs:IsA("GuiObject") then
            add(rs:GetPropertyChangedSignal("Visible"):Connect(function()
                ap.rel(true)
                if not rs.Visible then
                    ap.defer(0.2)
                end
            end))
        end

        add(root.ChildAdded:Connect(function(ch)
            ap.index(ch)
            if ap.ingame then
                return
            end

            if ch.Name == "GameplayFrame" or ch.Name == "FailedFrame" or ch.Name == "RankingScreen" then
                ap.defer(0.1)
            end
        end))

        add(root.ChildRemoved:Connect(function(ch)
            ap.dropui(ch)
        end))
    end
end

ap.setupwatch = function()
    ap.pg = ap.lp and ap.lp:FindFirstChildOfClass("PlayerGui")
    if not ap.pg then
        return
    end

    ap.primeui(false)

    local root = ap.findroot()

    if root then
        local menus = root:FindFirstChild("MENUS")
        local sel = menus and menus:FindFirstChild("SelectionFrame")
        local start = menus and menus:FindFirstChild("StartFrame")
        local msg = root:FindFirstChild("Message")

        if sel and sel:IsA("GuiObject") then
            ap.add(sel:GetPropertyChangedSignal("Visible"):Connect(function()
                if not sel.Visible then
                    ap.startscan()
                end
            end))
        end

        if start and start:IsA("GuiObject") then
            ap.add(start:GetPropertyChangedSignal("Visible"):Connect(function()
                if not start.Visible and (not sel or not sel.Visible) then
                    ap.startscan()
                end
            end))
        end

        if msg and msg:IsA("GuiObject") then
            ap.add(msg:GetPropertyChangedSignal("Visible"):Connect(function()
                if ap.ingame then
                    return
                end

                if not msg.Visible and (not sel or not sel.Visible) then
                    ap.g = nil
                    ap.inp = nil
                    ap.ctx = nil
                    ap.defer(0.05)
                end
            end))
        end
    end

    ap.add(ap.pg.ChildAdded:Connect(function(v)
        ap.index(v)
        if not ap.cfg.autorescan or ap.dead or ap.ingame then
            return
        end
        ap.defer(0.05)
    end))

    ap.add(ap.pg.ChildRemoved:Connect(function(v)
        ap.dropui(v)
    end))

    ap.add(ap.pg.DescendantAdded:Connect(function(v)
        ap.index(v)

        if not ap.cfg.autorescan or ap.dead or ap.ingame then
            return
        end

        local n = v.Name
        if n == "GameplayFrame" or n == "Tracks" or n == "TriggerButtons" or n == "FailedFrame" or n == "RankingScreen" then
            ap.defer(0.05)
        end
    end))

    ap.add(ap.pg.DescendantRemoving:Connect(function(v)
        ap.dropui(v)
    end))
end

ap.tap = function(i)
    if ap.hold[i] then
        return
    end

    ap.dn(i, true)

    if ap.down[i] then
        ap.relq[i] = ap.clock() + math.max(ap.cfg.tap, ap.cfg.pulse)
    end
end

ap.kind = function(n)
    local inf = ap.info(n)
    return inf and inf.k or nil
end

ap.scl = function(g)
    if g and g:IsA("GuiObject") then
        return g.Position.Y.Scale
    end
    return nil
end

ap.hd = function(n)
    local inf = ap.info(n)
    if not inf then
        return nil
    end

    local h = inf.h
    local t = inf.t
    local b = inf.b

    local hy = h and h:IsA("GuiObject") and ap.scl(h) or nil
    local ty = t and t:IsA("GuiObject") and ap.scl(t) or nil

    if hy or ty then
        return hy or ty, ty or hy, h or b or n, t or b or n, b
    end

    local y = ap.scl(n)
    return y, y, n, n, nil
end

ap.tar = function()
    return 1 - ap.cfg.lead
end

ap.was = function(n, y, span, life)
    local r = ap.seen[n]
    if not r then
        return false
    end

    if ap.clock() - r.t > (life or 0.35) then
        ap.seen[n] = nil
        return false
    end

    return math.abs((y or 0) - (r.y or 0)) <= (span or 0.2)
end

ap.mark = function(n, y)
    ap.seen[n] = {
        y = y or 0,
        t = ap.clock()
    }
end

ap.cross = function(n, y, target)
    if not n or type(y) ~= "number" or type(target) ~= "number" then
        return false
    end

    local old = ap.pos[n]
    ap.pos[n] = y

    if type(old) ~= "number" then
        return false
    end

    return (old < target and y >= target) or (old > target and y <= target)
end

ap.hready = function(n)
    local inf = n and ap.info(n) or nil
    local h = inf and inf.h or nil
    if h and h:IsA("GuiObject") then
        return h.Visible ~= false
    end
    return true
end

ap.tapWindow = function()
    return ap.cfg.precision and ap.cfg.precisionWindow or ap.cfg.win
end

ap.holdWindow = function()
    return ap.cfg.precision and ap.cfg.precisionWindow * 1.5 or ap.cfg.hwin
end

ap.tapok = function(n)
    local y = ap.scl(n)
    if not y then
        return false, math.huge
    end

    if ap.was(n, y, 0.18, 0.25) then
        return false, math.huge
    end

    local target = ap.tar()
    local crossed = ap.cross(n, y, target)
    local d = math.abs(y - target)
    return crossed or d <= ap.tapWindow(), d
end

ap.holdok = function(n)
    if not ap.hready(n) then
        return false, math.huge
    end

    local h = ap.hd(n)
    if not h then
        return false, math.huge
    end

    if ap.was(n, h, 0.18, 0.4) then
        return false, math.huge
    end

    local target = ap.tar()
    local crossed = ap.cross(n, h, target)
    local d = math.abs(h - target)
    return crossed or d <= ap.holdWindow(), d
end

ap.rtar = function()
    local v = 1 - ap.cfg.rlead
    if v > 1 then
        return 1
    end
    if v < 0 then
        return 0
    end
    return v
end

ap.hrelok = function(hd, now)
    local n = hd and hd.n
    if not n then
        return false
    end

    local _, t = ap.hd(n)
    if not t then
        return false
    end

    if not hd.lasttail then
        hd.lasttail = t
    end

    local target = ap.rtar()
    local crossed = t >= target or (hd.lasttail < target and t >= target)
    hd.lasttail = t

    if crossed then
        if ap.cfg.holdlate <= 0 then
            return true
        end
        hd.ready = hd.ready or now
        return now - hd.ready >= ap.cfg.holdlate
    end

    hd.ready = nil
    return false
end

ap.best = function(tr)
    local ht = nil
    local hd = math.huge
    local hc = false
    local tp = nil
    local td = math.huge
    local tc = false

    for _, n in ap.notes(tr) do
        local k = ap.kind(n)
        if k and ap.nvis(n) then
            if k == "hold" then
                local ok, d = ap.holdok(n)
                if (ok or d <= ap.holdWindow() * 3) and d < hd then
                    ht = n
                    hd = d
                    hc = ok
                end
            elseif k == "tap" then
                local ok, d = ap.tapok(n)
                if (ok or d <= ap.tapWindow() * 3) and d < td then
                    tp = n
                    td = d
                    tc = ok
                end
            end
        end
    end

    if ht and (hc or hd <= ap.holdWindow()) then
        return ht, "hold"
    end

    if tp and (tc or td <= ap.tapWindow()) then
        return tp, "tap"
    end
    return nil, nil
end

ap.sync = function()
    if ap.syncdone or not ap.cfg.new then
        return
    end

    ap.conf = ap.conf or ap.rs:FindFirstChild("Configuration")
    if not ap.conf then
        return
    end

    ap.useinp = ap.useinp or ap.conf:FindFirstChild("UseNewInput")
    local v = ap.useinp
    if v and v:IsA("BoolValue") then
        if v.Value == false then
            v.Value = true
        end
        ap.syncdone = true
    end
end

ap.step = function()
    if ap.dead or not ap.cfg.on then
        return
    end

    local now = ap.clock()
    local active = ap.g and ap.active(ap.g) or false

    if active then
        ap.ingame = true
        ap.needscan = nil
    else
        ap.ingame = false

        if ap.needscan and now >= ap.needscan then
            ap.needscan = nil
            if not ap.ready() then
                ap.defer(0.5)
                return
            end
            ap.reset(true)
            ap.g = ap.ui()
        elseif not ap.g or not ap.live(ap.g) then
            if now >= (ap.nextui or 0) then
                ap.nextui = now + 3
                if ap.ready() then
                    ap.rel(true)
                    ap.g = ap.ui()
                end
            end
        end

        active = ap.g and ap.active(ap.g) or false
    end

    if not active then
        ap.ingame = false
        if not ap.idle then
            ap.idle = true
            ap.rel(true)
        end
        return
    end

    ap.ingame = true
    ap.idle = false

    if ap.lastg ~= ap.g then
        ap.lastg = ap.g
        ap.reset(false)
        ap.watch(ap.g)
    end

    ap.sync()

    if not ap.trf or ap.trf.Parent ~= ap.g then
        ap.refreshtracks(ap.g)
    end

    local trf = ap.trf
    if not trf then
        return
    end

    for i = 1, 4 do
        local tr = ap.trk[i]

        if not tr or tr.Parent ~= trf then
            ap.refreshtracks(ap.g)
            tr = ap.trk[i]
        end

        if not tr then
            ap.up(i)
            continue
        end

        local hd = ap.hold[i]

        if hd then
            local hn = hd.n or hd

            if ap.nvis(hn) then
                ap.dn(i)

                if ap.hrelok(hd, now) then
                    local _, ty = ap.hd(hn)
                    ap.mark(hn, ty)
                    ap.up(i)
                    ap.dropnote(hn)
                end

                continue
            end

            local _, ty = ap.hd(hn)
            ap.mark(hn, ty)
            ap.up(i)
            ap.dropnote(hn)
        end

        local n, k = ap.best(tr)

        if n then
            if k == "hold" and ap.cfg.hold then
                local hy = ap.hd(n)
                ap.dn(i, true)
                if ap.down[i] then
                    ap.mark(n, hy)
                    ap.hold[i] = {
                        n = n,
                        ready = nil,
                        started = now
                    }
                end
            else
                local y = ap.scl(n)
                ap.mark(n, y)
                ap.tap(i)
                ap.dropnote(n)
            end
        elseif not ap.hold[i] then
            local rt = ap.relq[i]
            if not rt or now >= rt then
                ap.up(i)
            end
        end
    end
end

ap.ntf = function(title, message)
    if ap.window then
        pcall(function()
            ap.window:Notify({ title = title, content = message, duration = 3 })
        end)
    else
        warn("[Yvetzal Rosu] " .. tostring(message or title))
    end
end

ap.capmsg = function()
    local parts = {}
    parts[#parts + 1] = "input=VirtualInputManager"
    parts[#parts + 1] = "vim=" .. tostring(ap.vim ~= nil)
    parts[#parts + 1] = "loadstring=" .. tostring(type(loadstring) == "function")
    parts[#parts + 1] = "cloneref=" .. tostring(type(cloneref) == "function")
    parts[#parts + 1] = "gameenv=" .. tostring(type(ap.gameg()) == "table")
    return table.concat(parts, " | ")
end


ap.stop = function()
    if ap.dead then
        return
    end

    ap.dead = true
    ap.cfg.on = false

    pcall(function()
        ap.run:UnbindFromRenderStep(ap.bind)
    end)

    ap.rel(true)

    if ap.watchcon then
        for _, c in ap.watchcon do
            pcall(function()
                c:Disconnect()
            end)
        end
        ap.clear(ap.watchcon)
    end

    if type(ap.cleartracks) == "function" then
        ap.cleartracks()
    end

    for _, c in ap.con do
        pcall(function()
            c:Disconnect()
        end)
    end

    ap.clear(ap.con)
    ap.clear(ap.down)
    ap.clear(ap.hold)
    ap.clear(ap.relq)

    if ap.closeUi then
        ap.closeUi()
    end

    if getgenv and getgenv().Yvetzal_Rosu == ap then
        getgenv().Yvetzal_Rosu = nil
    end
end



local THEME = {
    WindowColor = ColorSequence.new(Color3.fromRGB(8, 8, 8), Color3.fromRGB(0, 0, 0)),
    ShadowColor = Color3.fromRGB(0, 0, 0),
    SurfaceStroke = Color3.fromRGB(120, 15, 15),
    CornerRoundness = UDim.new(0, 8),
    ElementCornerRadius = UDim.new(0, 6),
    ContentColor = Color3.fromRGB(230, 230, 230),
    TitlingColor = Color3.fromRGB(255, 255, 255),
    ActionColor = Color3.fromRGB(255, 255, 255),
    ElementTextHoverColor = Color3.fromRGB(255, 255, 255),
    TabColor = Color3.fromRGB(255, 255, 255),
    TabBackground = ColorSequence.new(Color3.fromRGB(22, 22, 22), Color3.fromRGB(8, 8, 8)),
    TabStroke = ColorSequence.new(Color3.fromRGB(230, 30, 30), Color3.fromRGB(110, 10, 10)),
    ElementGradient = ColorSequence.new(Color3.fromRGB(18, 18, 18), Color3.fromRGB(10, 10, 10)),
    ElementStroke = Color3.fromRGB(60, 60, 60),
    ElementStrokeHover = Color3.fromRGB(230, 40, 40),
    StatBackground = Color3.fromRGB(30, 6, 6),
    AccentColor = Color3.fromRGB(220, 25, 25),
    AccentStroke = Color3.fromRGB(255, 70, 70),
    SliderBackground = Color3.fromRGB(35, 35, 35),
    SliderProgress = ColorSequence.new(Color3.fromRGB(255, 255, 255), Color3.fromRGB(210, 210, 210)),
    SliderHandle = Color3.fromRGB(255, 255, 255),
    SliderStroke = Color3.fromRGB(60, 60, 60),
    ToggleTrack = Color3.fromRGB(30, 30, 30),
    ToggleKnobOff = Color3.fromRGB(220, 220, 220),
    DropdownHighlight = Color3.fromRGB(180, 25, 25),
    FieldBackground = Color3.fromRGB(14, 14, 14),
    FieldGlow = Color3.fromRGB(120, 20, 20),
    PlaceholderColor = Color3.fromRGB(150, 150, 150),
    NeutralButton = Color3.fromRGB(26, 26, 26),
    NeutralButtonHover = Color3.fromRGB(45, 45, 45),
    NeutralButtonStroke = Color3.fromRGB(80, 15, 15),
    ErrorColor = Color3.fromRGB(230, 40, 40),
    ErrorStrokeColor = Color3.fromRGB(255, 90, 90),
}

local function darkTheme(factor)
    local result = table.clone(THEME)
    for _, field in ipairs({ "WindowColor", "TabBackground", "ElementGradient" }) do
        local keypoints = {}
        for _, point in ipairs(THEME[field].Keypoints) do
            keypoints[#keypoints + 1] = ColorSequenceKeypoint.new(point.Time, point.Value:Lerp(Color3.new(0, 0, 0), factor))
        end
        result[field] = ColorSequence.new(keypoints)
    end
    for _, field in ipairs({ "SliderBackground", "ToggleTrack", "FieldBackground", "NeutralButton", "NeutralButtonHover" }) do
        result[field] = THEME[field]:Lerp(Color3.new(0, 0, 0), factor)
    end
    return result
end

local crimsonTheme = table.clone(THEME)
crimsonTheme.AccentColor = Color3.fromRGB(220, 20, 60)
crimsonTheme.AccentStroke = Color3.fromRGB(255, 70, 100)
local themes = { Default = THEME, Crimson = crimsonTheme, Dark = darkTheme(0.35), Darker = darkTheme(0.65) }
local themeName = "Default"
if getgenv and themes[getgenv().Infernum_Theme] then
    themeName = getgenv().Infernum_Theme
end

local ok, rayfield = pcall(function()
    return loadstring(game:HttpGet("https://sirius.menu/gen2"))()
end)
if not ok or not rayfield then
    ap.stop()
    warn("[Yvetzal Rosu] Rayfield Gen2 failed to load: " .. tostring(rayfield))
    return
end

local iconAtlas = {
    flame = { 16898613353, 967, 306 },
    music = { 16898613613, 967, 563 },
    settings = { 16898613777, 771, 257 },
    ["toggle-right"] = { 16898613869, 820, 98 },
}

local function icon(name)
    local sprite = iconAtlas[name]
    return sprite and ("rbxassetid://" .. sprite[1]) or name
end

local function crop(image, name)
    local sprite = iconAtlas[name]
    if image and sprite then
        image.ImageRectSize = Vector2.new(48, 48)
        image.ImageRectOffset = Vector2.new(sprite[2], sprite[3])
    end
end

local window = rayfield:CreateWindow({
    name = "Yvetzal",
    subtitle = "rosu!mania",
    icon = icon("flame"),
    showName = "Yvetzal",
    showIcon = icon("flame"),
    sidebarLayout = true,
    profile = "Wyrizon & The Infernum Team",
    theme = themes[themeName],
})
ap.window = window
crop(window.topbarIcon, "flame")
crop(window.collapsedIcon, "flame")

local rawUnload = window.Unload
ap.closeUi = function()
    if ap.uiClosed then
        return
    end
    ap.uiClosed = true
    pcall(rawUnload, window)
end
window.Unload = function()
    ap.stop()
end

local function createTab(name, iconName)
    local tab = window:CreateTab({ name = name, icon = icon(iconName) })
    crop(tab.topbarItemIcon, iconName)
    for _, method in ipairs({ "CreateSection", "CreateToggle", "CreateSlider", "CreateButton", "CreateText" }) do
        local create = tab[method]
        tab[method] = function(self, props)
            local controlIcon = props.icon
            if iconAtlas[controlIcon] then
                props.icon = icon(controlIcon)
            end
            local element = create(self, props)
            crop(element and element.iconLabel, controlIcon)
            return element
        end
    end
    return tab
end

local playTab = createTab("Autoplay", "music")
local settingsTab = createTab("Settings", "settings")

playTab:CreateSection({ name = "AUTOPLAY", icon = "toggle-right" })
playTab:CreateToggle({
    name = "Enable Autoplay",
    value = false,
    flag = "RosuAutoplay",
    callback = function(enabled)
        ap.cfg.on = enabled
        if not enabled then
            ap.rel(true)
        else
            ap.reset(true)
            ap.g = ap.ui()
        end
        ap.ntf("Autoplay", enabled and "Enabled" or "Disabled")
    end,
})
playTab:CreateToggle({
    name = "Hold Notes",
    value = ap.cfg.hold,
    flag = "RosuHoldNotes",
    callback = function(enabled)
        ap.cfg.hold = enabled
        if not enabled then
            ap.rel(true)
        end
    end,
})
playTab:CreateToggle({
    name = "Precision Mode",
    description = "Use a narrow trigger window around the hit line",
    value = ap.cfg.precision,
    flag = "RosuPrecision",
    callback = function(enabled)
        ap.cfg.precision = enabled
    end,
})
playTab:CreateSlider({
    name = "Precision Window",
    range = { 0.005, 0.05 },
    value = ap.cfg.precisionWindow,
    increment = 0.001,
    flag = "RosuPrecisionWindow",
    callback = function(value)
        ap.cfg.precisionWindow = value
    end,
})
playTab:CreateSlider({
    name = "Hit Lead",
    description = "Adjust if all notes register early or late",
    range = { -0.12, 0.12 },
    value = ap.cfg.lead,
    increment = 0.001,
    flag = "RosuHitLead",
    callback = function(value)
        ap.cfg.lead = value
    end,
})
playTab:CreateSlider({
    name = "Hold Tail Target",
    range = { 0, 0.12 },
    value = ap.cfg.rlead,
    increment = 0.001,
    flag = "RosuTailTarget",
    callback = function(value)
        ap.cfg.rlead = value
    end,
})

settingsTab:CreateSection({ name = "TIMING", icon = "settings" })
local sliders = {
    { "Tap Window", "RosuTapWindow", "win", 0.01, 0.18, 0.001 },
    { "Hold Start Window", "RosuHoldWindow", "hwin", 0.01, 0.16, 0.001 },
    { "Hold Extra Time", "RosuHoldLate", "holdlate", 0, 0.16, 0.001 },
    { "Tap Duration", "RosuTapDuration", "tap", 0.01, 0.18, 0.001 },
    { "Minimum Key Hold", "RosuKeyHold", "pulse", 0.01, 0.18, 0.001 },
}
for _, control in ipairs(sliders) do
    settingsTab:CreateSlider({
        name = control[1], flag = control[2], range = { control[4], control[5] },
        value = ap.cfg[control[3]], increment = control[6],
        callback = function(value)
            ap.cfg[control[3]] = value
        end,
    })
end

settingsTab:CreateSection({ name = "ACTIONS", icon = "settings" })
settingsTab:CreateButton({
    name = "Rescan Gameplay",
    callback = function()
        ap.reset(true)
        ap.g = ap.ui()
        ap.ntf("Autoplay", "Gameplay cache refreshed")
    end,
})
settingsTab:CreateButton({
    name = "Release Inputs",
    callback = function()
        ap.rel(true)
        ap.ntf("Autoplay", "Released all inputs")
    end,
})
settingsTab:CreateButton({
    name = "Unload Script",
    callback = function()
        window:Unload()
    end,
})
settingsTab:CreateText({ name = "Timing", text = "A perfect score depends on the chart, latency, and game judgement. Tune Hit Lead if needed." })

ap.setupwatch()
ap.defer(0.05)
ap.run:BindToRenderStep(ap.bind, Enum.RenderPriority.Input.Value, ap.step)
if getgenv then
    getgenv().Yvetzal_Rosu = ap
end
ap.ntf("Autoplay", "Ready | " .. ap.capmsg())
