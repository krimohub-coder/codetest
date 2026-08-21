--[[
    skyr0.wtf - code redeemer
    not by me ( solar ) btw just found in a server self leak
--]]

local cloneref = cloneref or function(o) return o end
local function svc(name)
    local ok, s = pcall(function() return game:GetService(name) end)
    if not ok or not s then return nil end
    local ok2, r = pcall(cloneref, s)
    if ok2 and r then return r end
    return s
end

local Players           = svc("Players")
local ReplicatedStorage  = svc("ReplicatedStorage")
local RunService         = svc("RunService")
local UserInputService   = svc("UserInputService")
local HttpService        = svc("HttpService")
local TweenService       = svc("TweenService")
local CoreGui            = svc("CoreGui")

local player = Players.LocalPlayer
if not player then
    Players:GetPropertyChangedSignal("LocalPlayer"):Wait()
    player = Players.LocalPlayer
end
local playerGui = player:WaitForChild("PlayerGui")

local env = (typeof(getgenv) == "function" and getgenv()) or _G

-- kill any previous instance of this script
if env.Skyr0Stop then pcall(env.Skyr0Stop) end
if env.StopAura then pcall(env.StopAura) end

local getupvalues = (debug and debug.getupvalues) or getupvalues
local getconns    = getconnections or (debug and debug.getconnections)
local setupv      = (debug and debug.setupvalue) or setupvalue

-- CONFIG
local CONFIG_FILE = "skyr0wtf_code_redeemer.json"
local cfg = {
    sniper        = true,
    autoSubmit    = true,
    submitAfter   = 3,
    retypeInvalid = false,
    riddleSolver  = true,
}

pcall(function()
    if type(isfile) == "function" and type(readfile) == "function" and isfile(CONFIG_FILE) then
        local decoded = HttpService:JSONDecode(readfile(CONFIG_FILE))
        if type(decoded) == "table" then
            for key, value in pairs(cfg) do
                local got = decoded[key]
                if type(got) == type(value) then cfg[key] = got end
            end
            cfg.submitAfter = math.max(1, math.floor(tonumber(cfg.submitAfter) or 3))
        end
    end
end)

local function saveConfig()
    if type(writefile) ~= "function" then return end
    pcall(function() writefile(CONFIG_FILE, HttpService:JSONEncode(cfg)) end)
end

-- STATE + FORWARD DECLARATIONS
local UI_NAME = "skyr0wtf_CodeRedeemer"

local _seen               = {}
local _capturedParts      = {}
local _lastBox            = nil
local _focused            = nil
local _lastWatchedBox     = nil
local _lastNonBlankText   = ""
local _boxTextConn, _boxAncestryConn
local _pendingText, _pendingBox, _pendingUntil, _pendingToken = nil, nil, 0, 0
local _solvedCount        = 0
local _askedCount         = 0
local _riddleQueue        = {}
local _riddleBusy         = false

local GUI                                  -- ScreenGui
local logRich, clearLog                     -- console writers
local setStatus
local typeAndSubmitCode, appendToBox, clearCapture
local rememberPending, clearPending, handleFeedback
local bumpSolvedLabel, applyPowerVisual

-- SMALL UTILITIES
local function trim(s) return (tostring(s or ""):gsub("^%s+", ""):gsub("%s+$", "")) end
local function stripRich(s)
    if type(s) ~= "string" then return tostring(s) end
    return (s:gsub("<[^>]->", ""))
end
local function upperClean(s)
    return (tostring(s or ""):upper():gsub("%s+", ""):gsub("[%?%.%,!\"'`\n\r]", ""))
end

-- SAB KNOWLEDGE BASE
local SAB_DB = {
    -- owner identity
    ["real name"]                   = "SAMMY",
    ["sammy real name"]             = "SAMMY",
    ["sammys real name"]            = "SAMMY",
    ["my real name"]                = "SAMMY",
    ["creator real name"]           = "SAMMY",
    ["owner real name"]             = "SAMMY",
    ["creator name"]                = "SAMMY",
    ["who created sab"]             = "SAMMY",
    ["who made sab"]                = "SAMMY",
    ["who made steal a brainrot"]   = "SAMMY",
    ["who is the owner"]            = "SAMMY",
    ["who owns sab"]                = "SAMMY",
    ["owner"]                       = "SAMMY",
    ["creator"]                     = "SAMMY",
    ["name"]                        = "SAMMY",
    ["my name"]                     = "SAMMY",
    ["sammy name"]                  = "SAMMY",
    ["sammys name"]                 = "SAMMY",
    ["game owner"]                  = "SAMMY",
    ["developer"]                   = "SAMMY",
    ["dev"]                         = "SAMMY",
    ["made by"]                     = "SAMMY",
    ["created by"]                  = "SAMMY",
    ["made this game"]              = "SAMMY",
    ["real name of sammy"]          = "SAMMY",
    ["roblox username"]             = "SPYDERSAMMY",
    ["my roblox username"]          = "SPYDERSAMMY",
    ["sammy username"]              = "SPYDERSAMMY",
    ["sammy roblox name"]           = "SPYDERSAMMY",
    ["roblox name"]                 = "SPYDERSAMMY",
    ["username"]                    = "SPYDERSAMMY",
    ["my username"]                 = "SPYDERSAMMY",
    ["roblox"]                      = "SPYDERSAMMY",
    ["channel"]                     = "SPYDERSAMMY",
    ["handle"]                      = "SPYDERSAMMY",

    -- age / birth
    ["how old am i"]                = "24",
    ["how old is sammy"]            = "24",
    ["my age"]                      = "24",
    ["sammy age"]                   = "24",
    ["age"]                         = "24",
    ["birth year"]                  = "2002",
    ["year born"]                   = "2002",
    ["year i was born"]             = "2002",
    ["born year"]                   = "2002",
    ["birth day"]                   = "FRIDAY",
    ["day i was born"]              = "FRIDAY",
    ["day born"]                    = "FRIDAY",
    ["birthday"]                    = "FRIDAY",
    ["born on"]                     = "FRIDAY",
    ["birth month"]                 = "FEBRUARY",
    ["month born"]                  = "FEBRUARY",
    ["month i was born"]            = "FEBRUARY",
    ["where was i born"]            = "ALGERIA",
    ["where was i born at"]         = "ALGERIA",
    ["birthplace"]                  = "ALGERIA",
    ["where i was born"]            = "ALGERIA",

    -- location
    ["where am i from"]             = "BRAZIL",
    ["where is sammy from"]         = "BRAZIL",
    ["my country"]                  = "BRAZIL",
    ["sammy country"]               = "BRAZIL",
    ["country"]                     = "BRAZIL",
    ["where do i live"]             = "BRAZIL",
    ["where does sammy live"]       = "BRAZIL",
    ["sammy location"]              = "BRAZIL",
    ["location"]                    = "BRAZIL",
    ["from"]                        = "BRAZIL",
    ["birth country"]               = "BRAZIL",
    ["born in"]                     = "BRAZIL",
    ["lives in"]                    = "BRAZIL",
    ["comes from"]                  = "BRAZIL",
    ["nationality"]                 = "BRAZILIAN",
    ["sammy nationality"]           = "BRAZILIAN",
    ["my nationality"]              = "BRAZILIAN",
    ["state"]                       = "SAOPAULO",
    ["my state"]                    = "SAOPAULO",
    ["sammy state"]                 = "SAOPAULO",
    ["city"]                        = "SAOPAULO",
    ["my city"]                     = "SAOPAULO",
    ["sammy city"]                  = "SAOPAULO",

    -- favourites
    ["favorite color"]              = "BLUE",
    ["my color"]                    = "BLUE",
    ["sammy color"]                 = "BLUE",
    ["color"]                       = "BLUE",
    ["favorite color is blue"]      = "BLUE",
    ["color is blue"]               = "BLUE",
    ["favorite sport"]              = "FOOTBALL",
    ["sport"]                       = "FOOTBALL",
    ["my sport"]                    = "FOOTBALL",
    ["football"]                    = "FOOTBALL",
    ["favorite football player"]    = "RONALDO",
    ["my favorite football player"] = "RONALDO",
    ["football player"]             = "RONALDO",
    ["favorite player"]             = "RONALDO",
    ["player"]                      = "RONALDO",
    ["ronaldo"]                     = "RONALDO",
    ["favorite food"]               = "PIZZA",
    ["my food"]                     = "PIZZA",
    ["food"]                        = "PIZZA",
    ["favorite meal"]               = "PIZZA",
    ["favorite dish"]               = "PIZZA",
    ["favorite animal"]             = "SPIDER",
    ["my animal"]                   = "SPIDER",
    ["my pet"]                      = "SPIDER",
    ["pet name"]                    = "SPIDER",
    ["sammy pet name"]              = "SPIDER",
    ["pet"]                         = "SPIDER",
    ["animal"]                      = "SPIDER",
    ["favorite game"]               = "ROBLOX",
    ["game"]                        = "ROBLOX",
    ["favorite number"]             = "SEVEN",
    ["lucky number"]                = "SEVEN",
    ["number"]                      = "SEVEN",
    ["social media"]                = "YOUTUBE",
    ["youtube channel"]             = "SPYDERSAMMY",
    ["my youtube"]                  = "SPYDERSAMMY",
    ["sammy youtube"]               = "SPYDERSAMMY",
    ["youtube"]                     = "SPYDERSAMMY",
    ["discord"]                     = "ACE",
    ["discord server"]              = "ACE",
    ["sammy discord"]               = "SPYDERSAMMY",
    ["twitter"]                     = "SPYDERSAMMY",
    ["sammy twitter"]               = "SPYDERSAMMY",
    ["x account"]                   = "SPYDERSAMMY",
    ["tiktok"]                      = "SPYDERSAMMY",
    ["sammy tiktok"]                = "SPYDERSAMMY",
    ["instagram"]                   = "SPYDERSAMMY",

    -- release info
    ["game created on"]             = "FRIDAY",
    ["created on"]                  = "FRIDAY",
    ["what day was the game created"] = "FRIDAY",
    ["what day was sab created"]    = "FRIDAY",
    ["game creation day"]           = "FRIDAY",
    ["release month"]               = "MAY",
    ["release year"]                = "2025",
    ["year sab was created"]        = "2025",
    ["what year was sab created"]   = "2025",
    ["what year was the game created"] = "2025",
    ["year the game was created"]   = "2025",
    ["year sab was made"]           = "2025",
    ["month sab was made"]          = "MAY",
    ["month the game was made"]     = "MAY",
    ["month sab was released"]      = "MAY",
    ["what month was sab made"]     = "MAY",
    ["what month was sab released"] = "MAY",
    ["sab release month"]           = "MAY",
    ["day sab was made"]            = "FRIDAY",
    ["day sab was released"]        = "FRIDAY",
    ["what day was sab made"]       = "FRIDAY",
    ["what day was sab released"]   = "FRIDAY",
    ["sab release day"]             = "FRIDAY",
    ["game made on"]                = "FRIDAY",
    ["sab made on"]                 = "FRIDAY",
    ["day game released"]           = "FRIDAY",
    ["day sab released"]            = "FRIDAY",
    ["release day"]                 = "FRIDAY",
    ["day released"]                = "FRIDAY",
    ["what day was it released"]    = "FRIDAY",
    ["what day was it created"]     = "FRIDAY",
    ["year the game was made"]      = "2025",
    ["year made"]                   = "2025",
    ["year created"]                = "2025",
    ["what year was sab made"]      = "2025",
    ["year of sab"]                 = "2025",
    ["sab creation year"]           = "2025",
    ["creation year"]               = "2025",
    ["when was sab made"]           = "MAY162025",
    ["when made"]                   = "MAY162025",
    ["date made"]                   = "MAY162025",
    ["when was sab created"]        = "MAY162025",
    ["release date"]                = "MAY162025",
    ["when was sab released"]       = "MAY162025",
    ["when was the game released"]  = "MAY162025",
    ["game release date"]           = "MAY162025",
    ["game release"]                = "MAY162025",
    ["sab release"]                 = "MAY162025",
    ["game released"]               = "MAY162025",
    ["sab released"]                = "MAY162025",
    ["when created"]                = "MAY162025",
    ["when released"]               = "MAY162025",
    ["date released"]               = "MAY162025",
    ["date created"]                = "MAY162025",

    -- repeats
    ["my name twice"]               = "SAMMYSAMMY",
    ["my name 2 times"]             = "SAMMYSAMMY",
    ["my name 3 times"]             = "SAMMYSAMMYSAMMY",
    ["name twice"]                  = "SAMMYSAMMY",
    ["owner twice"]                 = "SAMMYSAMMY",
    ["creator twice"]               = "SAMMYSAMMY",
    ["my age twice"]                = "2424",
    ["my age 2 times"]              = "2424",
    ["my age 3 times"]              = "242424",
    ["favorite color twice"]        = "BLUEBLUE",
    ["favorite color 2 times"]      = "BLUEBLUE",
    ["favorite color 3 times"]      = "BLUEBLUEBLUE",
    ["favorite color three times"]  = "BLUEBLUEBLUE",
    ["favorite color 5 times"]      = "BLUEBLUEBLUEBLUEBLUE",
    ["my favorite color twice"]     = "BLUEBLUE",
    ["my favorite color 2 times"]   = "BLUEBLUE",
    ["my favorite color 3 times"]   = "BLUEBLUEBLUE",
    ["favorite sport twice"]        = "FOOTBALLFOOTBALL",
    ["favorite food twice"]         = "PIZZAPIZZA",

    -- traits
    ["first trait"]                 = "LIGHTNING",
    ["1st trait"]                   = "LIGHTNING",
    ["first trait created"]         = "LIGHTNING",
    ["1st trait created"]           = "LIGHTNING",
    ["first trait added"]           = "LIGHTNING",
    ["trait added first"]           = "LIGHTNING",
    ["trait first added"]           = "LIGHTNING",
    ["what trait"]                  = "LIGHTNING",
    ["trait"]                       = "LIGHTNING",
    ["trait you get when struck by lightning"] = "MATEO",
    ["struck by lightning"]         = "MATEO",
    ["lightning trait"]             = "MATEO",
    ["trait from lightning"]        = "MATEO",
    ["trait when struck by lightning"] = "MATEO",
    ["lightning strike trait"]      = "MATEO",
    ["get struck by lightning"]     = "MATEO",
    ["lightning gives"]             = "MATEO",
    ["struck by lightning trait"]   = "MATEO",
    ["lightning"]                   = "MATEO",
    ["mateo"]                       = "MATEO",

    -- mutations
    ["first mutation"]              = "GOLD",
    ["1st mutation"]                = "GOLD",
    ["second mutation"]             = "DIAMOND",
    ["2nd mutation"]                = "DIAMOND",
    ["third mutation"]              = "BLOODROT",
    ["3rd mutation"]                = "BLOODROT",
    ["fourth mutation"]             = "RAINBOW",
    ["4th mutation"]                = "RAINBOW",
    ["fifth mutation"]              = "CANDY",
    ["5th mutation"]                = "CANDY",
    ["sixth mutation"]              = "LAVA",
    ["6th mutation"]                = "LAVA",
    ["seventh mutation"]            = "GALAXY",
    ["7th mutation"]                = "GALAXY",
    ["eighth mutation"]             = "YINYANG",
    ["8th mutation"]                = "YINYANG",
    ["ninth mutation"]              = "RADIOACTIVE",
    ["9th mutation"]                = "RADIOACTIVE",
    ["tenth mutation"]              = "CURSED",
    ["10th mutation"]               = "CURSED",
    ["eleventh mutation"]           = "DIVINE",
    ["11th mutation"]               = "DIVINE",
    ["twelfth mutation"]            = "CYBER",
    ["12th mutation"]               = "CYBER",
    ["thirteenth mutation"]         = "PHANTOM",
    ["13th mutation"]               = "PHANTOM",
    ["fourteenth mutation"]         = "CRYSTAL",
    ["14th mutation"]               = "CRYSTAL",
    ["mutation 1"]  = "GOLD",        ["mutation number 1"]  = "GOLD",
    ["mutation 2"]  = "DIAMOND",     ["mutation number 2"]  = "DIAMOND",
    ["mutation 3"]  = "BLOODROT",    ["mutation number 3"]  = "BLOODROT",
    ["mutation 4"]  = "RAINBOW",     ["mutation number 4"]  = "RAINBOW",
    ["mutation 5"]  = "CANDY",       ["mutation number 5"]  = "CANDY",
    ["mutation 6"]  = "LAVA",        ["mutation number 6"]  = "LAVA",
    ["mutation 7"]  = "GALAXY",      ["mutation number 7"]  = "GALAXY",
    ["mutation 8"]  = "YINYANG",     ["mutation number 8"]  = "YINYANG",
    ["mutation 9"]  = "RADIOACTIVE", ["mutation number 9"]  = "RADIOACTIVE",
    ["mutation 10"] = "CURSED",      ["mutation number 10"] = "CURSED",
    ["mutation 11"] = "DIVINE",      ["mutation number 11"] = "DIVINE",
    ["mutation 12"] = "CYBER",       ["mutation number 12"] = "CYBER",
    ["mutation 13"] = "PHANTOM",     ["mutation number 13"] = "PHANTOM",
    ["mutation 14"] = "CRYSTAL",     ["mutation number 14"] = "CRYSTAL",
    ["evil mutation"]               = "CURSED",
    ["evil"]                        = "CURSED",
    ["cursed mutation"]             = "CURSED",
    ["angelic mutation"]            = "DIVINE",
    ["angelic"]                     = "DIVINE",
    ["divine mutation"]             = "DIVINE",
    ["good mutation"]               = "DIVINE",
    ["best mutation"]               = "DIVINE",
    ["top mutation"]                = "DIVINE",
    ["latest mutation"]             = "CRYSTAL",
    ["most recent mutation"]        = "CRYSTAL",
    ["most recent"]                 = "CRYSTAL",
    ["newest mutation"]             = "CRYSTAL",
    ["last mutation"]               = "CRYSTAL",
    ["divinecursed"]                = "DIVINECURSED",
    ["curseddivine"]                = "CURSEDDIVINE",
    ["green mutation"]              = "RADIOACTIVE",
    ["turns green"]                 = "RADIOACTIVE",
    ["green"]                       = "RADIOACTIVE",
    ["purple mutation"]             = "GALAXY",
    ["turns purple"]                = "GALAXY",
    ["purple"]                      = "GALAXY",
    ["black and white mutation"]    = "YINYANG",
    ["black mutation"]              = "YINYANG",
    ["turns black"]                 = "YINYANG",
    ["black and white"]             = "YINYANG",
    ["yellow mutation"]             = "DIVINE",
    ["turns yellow"]                = "DIVINE",
    ["yellow"]                      = "DIVINE",
    ["red mutation"]                = "CURSED",
    ["turns red"]                   = "CURSED",
    ["red"]                         = "CURSED",
    ["orange mutation"]             = "LAVA",
    ["turns orange"]                = "LAVA",
    ["orange"]                      = "LAVA",
    ["blue mutation"]               = "DIAMOND",
    ["pink mutation"]               = "CANDY",
    ["gold"]                        = "GOLD",
    ["diamond"]                     = "DIAMOND",
    ["bloodrot"]                    = "BLOODROT",
    ["rainbow"]                     = "RAINBOW",
    ["candy"]                       = "CANDY",
    ["lava"]                        = "LAVA",
    ["galaxy"]                      = "GALAXY",
    ["yinyang"]                     = "YINYANG",
    ["radioactive"]                 = "RADIOACTIVE",
    ["cursed"]                      = "CURSED",
    ["divine"]                      = "DIVINE",
    ["cyber"]                       = "CYBER",
    ["phantom"]                     = "PHANTOM",
    ["crystal"]                     = "CRYSTAL",
    ["color of gold mutation"]      = "YELLOW",
    ["color of diamond mutation"]   = "BLUE",
    ["color of bloodrot mutation"]  = "RED",
    ["color of rainbow mutation"]   = "RAINBOW",
    ["color of candy mutation"]     = "PINK",
    ["color of lava mutation"]      = "ORANGE",
    ["color of galaxy mutation"]    = "PURPLE",
    ["color of yinyang mutation"]   = "BLACK",
    ["color of radioactive mutation"] = "GREEN",
    ["color of cursed mutation"]    = "RED",
    ["color of divine mutation"]    = "YELLOW",
    ["color of cyber mutation"]     = "BLUE",
    ["color of phantom mutation"]   = "BLACK",
    ["color of crystal mutation"]   = "BLUEPURPLE",
    ["divine color"]                = "YELLOW",
    ["cursed color"]                = "RED",
    ["radioactive color"]           = "GREEN",
    ["galaxy color"]                = "PURPLE",
    ["yinyang color"]               = "BLACK",
    ["lava color"]                  = "ORANGE",
    ["what number is gold"]         = "1",
    ["what number is diamond"]      = "2",
    ["what number is bloodrot"]     = "3",
    ["what number is rainbow"]      = "4",
    ["what number is candy"]        = "5",
    ["what number is lava"]         = "6",
    ["what number is galaxy"]       = "7",
    ["what number is yinyang"]      = "8",
    ["what number is radioactive"]  = "9",
    ["what number is cursed"]       = "10",
    ["what number is divine"]       = "11",
    ["what number is cyber"]        = "12",
    ["what number is phantom"]      = "13",
    ["what number is crystal"]      = "14",

    -- machines
    ["first machine"]               = "RAINBOWMACHINE",
    ["1st machine"]                 = "RAINBOWMACHINE",
    ["second machine"]              = "BUBBLEGUMMACHINE",
    ["2nd machine"]                 = "BUBBLEGUMMACHINE",
    ["third machine"]               = "FUSEMACHINE",
    ["3rd machine"]                 = "FUSEMACHINE",
    ["fourth machine"]              = "CRAFTMACHINE",
    ["4th machine"]                 = "CRAFTMACHINE",
    ["fifth machine"]               = "WITCHFUSE",
    ["5th machine"]                 = "WITCHFUSE",
    ["sixth machine"]               = "BRAINROTDEALER",
    ["6th machine"]                 = "BRAINROTDEALER",
    ["seventh machine"]             = "BRAINROTTRADER",
    ["7th machine"]                 = "BRAINROTTRADER",
    ["eighth machine"]              = "SANTASFUSE",
    ["8th machine"]                 = "SANTASFUSE",
    ["ninth machine"]               = "SANTASSHOP",
    ["9th machine"]                 = "SANTASSHOP",
    ["tenth machine"]               = "NEWYEARSMACHINE",
    ["10th machine"]                = "NEWYEARSMACHINE",
    ["eleventh machine"]            = "DUELSMACHINE",
    ["11th machine"]                = "DUELSMACHINE",
    ["twelfth machine"]             = "CUPIDSMACHINE",
    ["12th machine"]                = "CUPIDSMACHINE",
    ["thirteenth machine"]          = "TRADEMACHINE",
    ["13th machine"]                = "TRADEMACHINE",
    ["fourteenth machine"]          = "DIVINEFUSE",
    ["14th machine"]                = "DIVINEFUSE",
    ["fifteenth machine"]           = "EGGINCUBATOR",
    ["15th machine"]                = "EGGINCUBATOR",
    ["sixteenth machine"]           = "CYBERCRAFTMACHINE",
    ["16th machine"]                = "CYBERCRAFTMACHINE",
    ["seventeenth machine"]         = "SUMMERFUSE",
    ["17th machine"]                = "SUMMERFUSE",
    ["eighteenth machine"]          = "LOSTRADERS",
    ["18th machine"]                = "LOSTRADERS",
    ["machine 1"]  = "RAINBOWMACHINE",    ["machine number 1"]  = "RAINBOWMACHINE",
    ["machine 2"]  = "BUBBLEGUMMACHINE",  ["machine number 2"]  = "BUBBLEGUMMACHINE",
    ["machine 3"]  = "FUSEMACHINE",       ["machine number 3"]  = "FUSEMACHINE",
    ["machine 4"]  = "CRAFTMACHINE",      ["machine number 4"]  = "CRAFTMACHINE",
    ["machine 5"]  = "WITCHFUSE",         ["machine number 5"]  = "WITCHFUSE",
    ["machine 6"]  = "BRAINROTDEALER",    ["machine number 6"]  = "BRAINROTDEALER",
    ["machine 7"]  = "BRAINROTTRADER",    ["machine number 7"]  = "BRAINROTTRADER",
    ["machine 8"]  = "SANTASFUSE",        ["machine number 8"]  = "SANTASFUSE",
    ["machine 9"]  = "SANTASSHOP",        ["machine number 9"]  = "SANTASSHOP",
    ["machine 10"] = "NEWYEARSMACHINE",   ["machine number 10"] = "NEWYEARSMACHINE",
    ["machine 11"] = "DUELSMACHINE",      ["machine number 11"] = "DUELSMACHINE",
    ["machine 12"] = "CUPIDSMACHINE",     ["machine number 12"] = "CUPIDSMACHINE",
    ["machine 13"] = "TRADEMACHINE",      ["machine number 13"] = "TRADEMACHINE",
    ["machine 14"] = "DIVINEFUSE",        ["machine number 14"] = "DIVINEFUSE",
    ["machine 15"] = "EGGINCUBATOR",      ["machine number 15"] = "EGGINCUBATOR",
    ["machine 16"] = "CYBERCRAFTMACHINE", ["machine number 16"] = "CYBERCRAFTMACHINE",
    ["machine 17"] = "SUMMERFUSE",        ["machine number 17"] = "SUMMERFUSE",
    ["machine 18"] = "LOSTRADERS",        ["machine number 18"] = "LOSTRADERS",
    ["rainbowmachine"]              = "RAINBOWMACHINE",
    ["bubblegummachine"]            = "BUBBLEGUMMACHINE",
    ["fusemachine"]                 = "FUSEMACHINE",
    ["craftmachine"]                = "CRAFTMACHINE",
    ["witchfuse"]                   = "WITCHFUSE",
    ["brainrotdealer"]              = "BRAINROTDEALER",
    ["brainrottrader"]              = "BRAINROTTRADER",
    ["santasfuse"]                  = "SANTASFUSE",
    ["santasshop"]                  = "SANTASSHOP",
    ["newyearsmachine"]             = "NEWYEARSMACHINE",
    ["duelsmachine"]                = "DUELSMACHINE",
    ["cupidsmachine"]               = "CUPIDSMACHINE",
    ["trademachine"]                = "TRADEMACHINE",
    ["divinefuse"]                  = "DIVINEFUSE",
    ["eggincubator"]                = "EGGINCUBATOR",
    ["cybercraftmachine"]           = "CYBERCRAFTMACHINE",
    ["summerfuse"]                  = "SUMMERFUSE",
    ["lostraders"]                  = "LOSTRADERS",
    ["newest machine"]              = "LOSTRADERS",
    ["last machine"]                = "LOSTRADERS",
    ["latest machine"]              = "LOSTRADERS",

    -- brainrots + rarities
    ["og brainrot cannot be obtained"] = "HEADLESSHORSEMAN",
    ["headless horseman"]           = "HEADLESSHORSEMAN",
    ["rarest brainrot"]             = "HEADLESSHORSEMAN",
    ["rarest"]                      = "HEADLESSHORSEMAN",
    ["unobtainable brainrot"]       = "HEADLESSHORSEMAN",
    ["unobtainable"]                = "HEADLESSHORSEMAN",
    ["best brainrot"]               = "STRAWBERRYELEPHANT",
    ["first og added"]              = "STRAWBERRYELEPHANT",
    ["1st og"]                      = "STRAWBERRYELEPHANT",
    ["second og added"]             = "MEOWL",
    ["2nd og"]                      = "MEOWL",
    ["third og added"]              = "SKIBIDITOILET",
    ["3rd og"]                      = "SKIBIDITOILET",
    ["fifth og added"]              = "JOHNPORK",
    ["5th og"]                      = "JOHNPORK",
    ["og 1"]  = "STRAWBERRYELEPHANT", ["og number 1"] = "STRAWBERRYELEPHANT",
    ["og 2"]  = "MEOWL",              ["og number 2"] = "MEOWL",
    ["og 3"]  = "SKIBIDITOILET",      ["og number 3"] = "SKIBIDITOILET",
    ["og 5"]  = "JOHNPORK",           ["og number 5"] = "JOHNPORK",
    ["first brainrot added"]        = "STRAWBERRYELEPHANT",
    ["1st brainrot"]                = "STRAWBERRYELEPHANT",
    ["oldest brainrot"]             = "STRAWBERRYELEPHANT",
    ["worst brainrot"]              = "NOOBINIPIZZANINI",
    ["most common brainrot"]        = "NOOBINIPIZZANINI",
    ["least rare brainrot"]         = "NOOBINIPIZZANINI",
    ["weakest brainrot"]            = "NOOBINIPIZZANINI",
    ["common brainrot"]             = "NOOBINIPIZZANINI",
    ["most popular brainrot"]       = "DRAGONCANNELONNI",
    ["highest rarity"]              = "OG",
    ["rarest rarity"]               = "OG",
    ["top rarity"]                  = "OG",
    ["best rarity"]                 = "OG",
    ["6th rarity"]                  = "OG",
    ["sixth rarity"]                = "OG",
    ["lowest rarity"]               = "COMMON",
    ["worst rarity"]                = "COMMON",
    ["common rarity"]               = "COMMON",
    ["1st rarity"]                  = "COMMON",
    ["second rarity"]               = "UNCOMMON",
    ["2nd rarity"]                  = "UNCOMMON",
    ["third rarity"]                = "RARE",
    ["3rd rarity"]                  = "RARE",
    ["fourth rarity"]               = "EPIC",
    ["4th rarity"]                  = "EPIC",
    ["fifth rarity"]                = "LEGENDARY",
    ["5th rarity"]                  = "LEGENDARY",
    ["uncommon"]                    = "UNCOMMON",
    ["rare"]                        = "RARE",
    ["epic"]                        = "EPIC",
    ["legendary"]                   = "LEGENDARY",
    ["og"]                          = "OG",
    ["common"]                      = "COMMON",

    -- trivia
    ["fire represents"]             = "DRAGON",
    ["fire stands for"]             = "DRAGON",
    ["fire symbol"]                 = "DRAGON",
    ["fire meaning"]                = "DRAGON",
    ["fire brainrot"]               = "DRAGON",
    ["fire"]                        = "DRAGON",
    ["dragon"]                      = "DRAGON",
    ["won the world cup"]           = "SPAIN",
    ["world cup winner"]            = "SPAIN",
    ["world cup"]                   = "SPAIN",
    ["won world cup"]               = "SPAIN",
    ["football world cup"]          = "SPAIN",
    ["worst game owner"]            = "SECRETLOKII",
    ["most boring game owner"]      = "SECRETLOKII",
    ["worst owner"]                 = "SECRETLOKII",
    ["boring owner"]                = "SECRETLOKII",
    ["most boring owner"]           = "SECRETLOKII",
    ["most boring game on roblox"]  = "KEYBOARDESCAPE",
    ["boring game"]                 = "KEYBOARDESCAPE",
    ["most boring game"]            = "KEYBOARDESCAPE",
    ["boring roblox game"]          = "KEYBOARDESCAPE",
    ["spawned during admin abuse war"] = "RACOONINIJANDELINI",
    ["admin war brainrot"]          = "RACOONINIJANDELINI",
    ["spawned in admin war"]        = "RACOONINIJANDELINI",
    ["who did i fight in the admin abuse war"] = "JANDEL",
    ["fought in admin abuse war"]   = "JANDEL",
    ["who did sammy fight"]         = "JANDEL",
    ["sammy fought"]                = "JANDEL",
    ["fight in admin war"]          = "JANDEL",
    ["won the admin abuse war"]     = "GROWAGARDEN",
    ["admin abuse war"]             = "GROWAGARDEN",
    ["who won admin war"]           = "GROWAGARDEN",
    ["admin war winner"]            = "GROWAGARDEN",
    ["admin war"]                   = "GROWAGARDEN",
    ["worst secret"]                = "KARKERKARKURKUR",
    ["secret"]                      = "KARKERKARKURKUR",
    ["bad secret"]                  = "KARKERKARKURKUR",
    ["maximum server size"]         = "EIGHT",
    ["max server size"]             = "EIGHT",
    ["server size"]                 = "EIGHT",
    ["how many players"]            = "EIGHT",
    ["max players"]                 = "EIGHT",
    ["players"]                     = "EIGHT",
    ["player count"]                = "EIGHT",
    ["server"]                      = "EIGHT",
    ["how many players in server"]  = "EIGHT",
    ["server capacity"]             = "EIGHT",
    ["players per server"]          = "EIGHT",
    ["max server"]                  = "EIGHT",
    ["brother of hydra bunny"]      = "CERBERUS",
    ["hydra bunny brother"]         = "CERBERUS",
    ["hydra bunny"]                 = "CERBERUS",
    ["cerberus brother"]            = "CERBERUS",
    ["brother of hydra"]            = "CERBERUS",
    ["brother"]                     = "CERBERUS",
    ["cerberus"]                    = "CERBERUS",

    -- game meta
    ["game name"]                   = "STEALABRAINROT",
    ["name of the game"]            = "STEALABRAINROT",
    ["sab"]                         = "STEALABRAINROT",
    ["sab stands for"]              = "STEALABRAINROT",
    ["full name"]                   = "STEALABRAINROT",
    ["what is sab"]                 = "STEALABRAINROT",
    ["steal a brainrot"]            = "STEALABRAINROT",
    ["full game name"]              = "STEALABRAINROT",
    ["game full name"]              = "STEALABRAINROT",
    ["sab full name"]               = "STEALABRAINROT",
    ["number of mutations"]         = "14",
    ["how many mutations"]          = "14",
    ["total mutations"]             = "THIRTEEN",
    ["mutation count"]              = "THIRTEEN",
    ["number of machines"]          = "18",
    ["how many machines"]           = "18",
    ["total machines"]              = "18",
    ["machine count"]               = "EIGHTEEN",
    ["total rarities"]              = "SIX",
    ["how many rarities"]           = "SIX",
    ["number of rarities"]          = "SIX",
    ["rarities"]                    = "SIX",
    ["rarity count"]                = "SIX",
    ["brainrot count"]              = "THIRTEEN",
    ["how many brainrots"]          = "THIRTEEN",
    ["number brainrots"]            = "THIRTEEN",
    ["how many og brainrots"]       = "FIVE",
    ["total og brainrots"]          = "FIVE",
    ["og count"]                    = "FIVE",
    ["type of game"]                = "SIMULATOR",
    ["game genre"]                  = "SIMULATOR",
    ["genre"]                       = "SIMULATOR",
    ["game type"]                   = "SIMULATOR",
    ["what type"]                   = "SIMULATOR",
    ["sab genre"]                   = "SIMULATOR",
    ["sab type"]                    = "SIMULATOR",
    ["first update"]                = "MUTATIONS",
    ["1st update"]                  = "MUTATIONS",
    ["update"]                      = "MUTATIONS",

    -- pre-baked combos (splitter handles the rest)
    ["favorite color favorite sport"]   = "BLUEFOOTBALL",
    ["favorite color favorite animal"]  = "BLUESPIDER",
    ["favorite color favorite food"]    = "BLUEPIZZA",
    ["favorite color favorite player"]  = "BLUERONALDO",
    ["favorite color creator"]          = "BLUESAMMY",
    ["favorite color owner"]            = "BLUESAMMY",
    ["favorite color username"]         = "BLUESPYDERSAMMY",
    ["favorite sport favorite player"]  = "FOOTBALLRONALDO",
    ["favorite sport favorite animal"]  = "FOOTBALLSPIDER",
    ["name favorite food"]              = "SAMMYPIZZA",
    ["name favorite color"]             = "SAMMYBLUE",
}

-- QUESTION NORMALISATION + LOCAL LOOKUP
local WORD_ALIAS = {
    favourite = "favorite", fav = "favorite", favorites = "favorite",
    colour = "color", colours = "color", colors = "color",
    whats = "what is", ["what's"] = "what is",
    whos = "who is", ["who's"] = "who is",
    wheres = "where is", whens = "when is",
    u = "you", ur = "your", pls = "", please = "",
    ["sammy's"] = "sammys", ["i'm"] = "i am",
}

local FILLERS = {
    "what is", "what are", "what was", "what were", "what do", "what does",
    "who is", "who was", "who are", "when is", "when was", "where is",
    "where are", "how old", "how many", "how much", "do you know",
    "can you tell me", "can you tell", "tell me", "i need", "give me",
    "answer me", "say", "type", "write", "the answer to", "question",
    "am i", "do i", "did i", "have i", "is my", "is the", "is a",
}

local NUMWORDS = {
    first = 1, second = 2, third = 3, fourth = 4, fifth = 5, sixth = 6,
    seventh = 7, eighth = 8, ninth = 9, tenth = 10, eleventh = 11,
    twelfth = 12, thirteenth = 13, fourteenth = 14, fifteenth = 15,
    sixteenth = 16, seventeenth = 17, eighteenth = 18,
}
local CARDINALS = {
    one = 1, two = 2, three = 3, four = 4, five = 5, six = 6, seven = 7,
    eight = 8, nine = 9, ten = 10, eleven = 11, twelve = 12,
    thirteen = 13, fourteen = 14, fifteen = 15, sixteen = 16,
    seventeen = 17, eighteen = 18,
}
local ORD_SUFFIX = { [1] = "1st", [2] = "2nd", [3] = "3rd" }
local function ordinal(n) return ORD_SUFFIX[n] or (tostring(n) .. "th") end

local function applyAliases(s)
    local out = {}
    for word in s:gmatch("%S+") do
        local swapped = WORD_ALIAS[word]
        if swapped ~= nil then
            if swapped ~= "" then out[#out + 1] = swapped end
        else
            out[#out + 1] = word
        end
    end
    return table.concat(out, " ")
end

local function basicClean(s)
    s = tostring(s or ""):lower()
    s = s:gsub("[%?%!%.%,%;%:\"'`]", " ")
    s = s:gsub("%s+", " ")
    return trim(s)
end

local function stripFillers(s)
    local out = " " .. s .. " "
    for _, filler in ipairs(FILLERS) do
        out = out:gsub("%s" .. filler:gsub("%%", "%%%%") .. "%s", " ")
    end
    out = out:gsub("%s+my%s+", " "):gsub("^%s*my%s+", " ")
    out = out:gsub("%s+sammys?%s+", " "):gsub("^%s*sammys?%s+", " ")
    out = out:gsub("%s+the%s+", " "):gsub("%s+of%s+", " "):gsub("%s+for%s+", " ")
    out = out:gsub("%s+a%s+", " "):gsub("%s+an%s+", " "):gsub("%s+in%s+", " ")
    out = out:gsub("%s+i%s+", " "):gsub("%s+is%s+", " "):gsub("%s+was%s+", " ")
    out = out:gsub("%s+", " ")
    return trim(out)
end

local function toDigits(s)
    local conv = s
    for word, n in pairs(NUMWORDS) do
        conv = conv:gsub("%f[%a]" .. word .. "%f[%A]", ordinal(n))
    end
    for word, n in pairs(CARDINALS) do
        conv = conv:gsub("%f[%a]" .. word .. "%f[%A]", tostring(n))
    end
    return conv
end

-- leading question words + the category noun right after them
local QWORDS = {
    who = true, what = true, which = true, when = true, where = true,
    how = true, why = true, name = true, tell = true, say = true,
    type = true, guess = true, answer = true, spell = true, write = true,
}
local CATEGORY_WORDS = {
    mutation = true, machine = true, trait = true, brainrot = true,
    rarity = true, color = true, number = true, thing = true, one = true,
    word = true, day = true, year = true, month = true, update = true,
}
local STOPWORDS = {
    you = true, your = true, yours = true, me = true, we = true, us = true,
    it = true, its = true, be = true, been = true, this = true, that = true,
    ["do"] = true, does = true, did = true, ["then"] = true,
}

local function words(s)
    local list = {}
    for word in s:gmatch("%S+") do list[#list + 1] = word end
    return list
end

-- drops a leading "what" / "what mutation" style opener
local function dropLead(s)
    local list = words(s)
    if #list < 2 or not QWORDS[list[1]] then return nil end
    table.remove(list, 1)
    if #list > 1 and CATEGORY_WORDS[list[1]] then table.remove(list, 1) end
    return table.concat(list, " ")
end

local function dropStopwords(s)
    local out = {}
    for _, word in ipairs(words(s)) do
        if not STOPWORDS[word] then out[#out + 1] = word end
    end
    return table.concat(out, " ")
end

-- Only returns a value when a normalised form of the question matches a key
-- exactly. No match means no answer.
local function localAnswer(text)
    if not text or text == "" then return nil end

    local raw      = basicClean(text)
    local aliased  = applyAliases(raw)
    local core     = stripFillers(aliased)
    local noStop   = dropStopwords(core)

    local candidates = { raw, aliased, core, noStop }

    local lead = dropLead(core)
    if lead then candidates[#candidates + 1] = lead end
    local leadNoStop = dropLead(noStop)
    if leadNoStop then candidates[#candidates + 1] = leadNoStop end

    -- written numbers -> digits, and the digit-free variant
    local base = #candidates
    for index = 1, base do
        local converted = toDigits(candidates[index])
        if converted ~= candidates[index] then candidates[#candidates + 1] = converted end
    end
    candidates[#candidates + 1] = (core:gsub("%d+", ""):gsub("%s+", " "))

    -- "mutation 5" <-> "5th mutation" in both directions
    local digitForm = toDigits(noStop)
    local kind, num = digitForm:match("^(.-)%s+(%d+)$")
    if kind and num then
        candidates[#candidates + 1] = ordinal(tonumber(num)) .. " " .. trim(kind)
    end
    local num2, kind2 = digitForm:match("^(%d+)%s+(.-)$")
    if num2 and kind2 then
        candidates[#candidates + 1] = ordinal(tonumber(num2)) .. " " .. trim(kind2)
    end

    for _, key in ipairs(candidates) do
        key = trim(key)
        if key ~= "" and SAB_DB[key] then return SAB_DB[key] end
    end
    return nil
end

-- RIDDLE ENGINE, multi question splitting
-- phrases that legitimately contain "and" and must not be split
local PROTECTED = {
    ["black and white"] = "\1blackwhite\1",
    ["yin and yang"]    = "\1yinyang\1",
    ["salt and pepper"] = "\1saltpepper\1",
    ["rock and roll"]   = "\1rockroll\1",
    ["cat and dog"]     = "\1catdog\1",
}
local UNPROTECT = {}
for phrase, token in pairs(PROTECTED) do UNPROTECT[token] = phrase end

local function splitQuestions(text)
    local s = " " .. tostring(text or ""):lower() .. " "
    for phrase, token in pairs(PROTECTED) do s = s:gsub(phrase, token) end

    -- punctuation separators FIRST, before basicClean wipes them
    s = s:gsub("%s*[%?%;%,]%s*", " and ")
    s = s:gsub("%s*&%s*", " and ")
    s = s:gsub("%s*%+%s*", " and ")
    s = " " .. basicClean(s) .. " "

    -- every remaining separator becomes " and "
    s = s:gsub("%s+add%s+", " and ")
    s = s:gsub("%s+plus%s+", " and ")
    s = s:gsub("%s+then%s+", " and ")
    s = s:gsub("%s+followed%s+by%s+", " and ")
    s = s:gsub("%s+also%s+", " and ")
    s = s:gsub("%s+", " ")

    local parts = {}
    for part in (s .. " and "):gmatch("(.-)%s+and%s+") do
        part = trim(part)
        if part ~= "" then
            for token, phrase in pairs(UNPROTECT) do part = part:gsub(token, phrase) end
            parts[#parts + 1] = part
        end
    end
    if #parts == 0 then parts[1] = trim(basicClean(text)) end
    return parts
end

local function multiplierOf(text)
    local l = text:lower()
    if l:find("twice") then return 2 end
    if l:find("thrice") then return 3 end
    local n = tonumber(l:match("(%d+)%s*times"))
    if n and n >= 2 and n <= 10 then return n end
    for word, value in pairs(CARDINALS) do
        if value >= 2 and value <= 10 and l:find("%f[%a]" .. word .. "%s+times") then return value end
    end
    return nil
end

local function stripMultiplier(text)
    local s = text
    s = s:gsub("%s*%d+%s*times%s*", " ")
    s = s:gsub("%s*twice%s*", " ")
    s = s:gsub("%s*thrice%s*", " ")
    for word, value in pairs(CARDINALS) do
        if value >= 2 and value <= 10 then s = s:gsub("%s*" .. word .. "%s+times%s*", " ") end
    end
    s = trim(s)
    return s ~= "" and s or text
end

-- a bare number the player must append, e.g. "favorite color and 67"
local function trailingSuffixNumber(text)
    local words, prev, suffix = {}, nil, nil
    for word in basicClean(text):gmatch("%S+") do words[#words + 1] = word end
    local SKIP = { mutation = true, machine = true, og = true, number = true,
                   brainrot = true, trait = true, rarity = true, times = true }
    for index, word in ipairs(words) do
        if word:match("^%d+$") and tonumber(word) > 13 then
            prev = index > 1 and words[index - 1] or ""
            local nxt = words[index + 1] or ""
            if not SKIP[prev] and not SKIP[nxt] then suffix = word end
        end
    end
    return suffix
end

local function stripTrailingNumber(text)
    return trim(tostring(text):gsub("%s+%d+%s*$", ""))
end

-- "my name at the end" -> the answer for that part goes last
local function extractPlacement(part)
    local placement
    if part:find("at the end") then placement = "end" end
    if part:find("at the start") or part:find("at the beginning") then placement = "start" end
    local cleaned = part
        :gsub("%s*at the end%s*", " ")
        :gsub("%s*at the start%s*", " ")
        :gsub("%s*at the beginning%s*", " ")
    return trim(cleaned), placement
end

-- database only. returns answer or nil.
local function resolvePart(part)
    local times = multiplierOf(part)
    local q = stripMultiplier(part)
    -- only strip a trailing number when it is a "tack this on" number,
    -- never when it identifies the thing being asked ("mutation 5")
    if trailingSuffixNumber(part) then q = stripTrailingNumber(q) end
    q = trim(q)
    if q == "" then return nil end

    -- a pure number is its own answer
    local onlyNumber = q:match("^%d+$")
    if onlyNumber then
        return times and string.rep(onlyNumber, times) or onlyNumber
    end

    local answer = localAnswer(q)
    if not answer then return nil end

    local clean = upperClean(answer)
    if clean == "" or clean == "AND" or clean:find("ATTHEEND") or clean:find("ATTHESTART") then
        return nil
    end
    return times and string.rep(clean, times) or clean
end

-- returns answer, breakdown(list), errMessage
local function solveRiddle(text)
    local suffix = trailingSuffixNumber(text)
    local parts = splitQuestions(text)
    local breakdown = {}
    local head, middle, tail = {}, {}, {}
    local missing = 0

    for _, rawPart in ipairs(parts) do
        local part, placement = extractPlacement(rawPart)
        local ok, result = pcall(resolvePart, part)
        if ok and result and result ~= "" then
            local bucket = (placement == "start" and head) or (placement == "end" and tail) or middle
            bucket[#bucket + 1] = result
            breakdown[#breakdown + 1] = part:sub(1, 30) .. " = " .. result
        else
            -- show what the database could not answer so it can be added
            missing = missing + 1
            breakdown[#breakdown + 1] = part:sub(1, 30) .. " = ?"
        end
    end

    local answers = {}
    for _, list in ipairs({ head, middle, tail }) do
        for _, value in ipairs(list) do answers[#answers + 1] = value end
    end

    if #answers == 0 then
        return nil, breakdown, "not in database"
    end

    local out = table.concat(answers, "")
    if suffix and out:sub(-#suffix) ~= suffix then out = out .. suffix end
    return out, breakdown, (missing > 0) and (missing .. (missing == 1 and " part" or " parts") .. " not in database") or nil
end

local QUESTION_STARTERS = {
    "what", "whats", "who", "whos", "when", "where", "which", "how",
    "name", "tell", "type", "say", "guess", "answer", "riddle", "spell",
}

local function looksLikeRiddle(text)
    local l = basicClean(text)
    if l == "" then return false end
    if not l:find(" ") then return false end
    if l:find("?", 1, true) then return true end
    local firstWord = l:match("^(%S+)")
    for _, starter in ipairs(QUESTION_STARTERS) do
        if firstWord == starter then return true end
    end
    -- "my name and my favorite food" style with no question word
    if l:find("%f[%a]my%f[%A]") and (l:find("%f[%a]and%f[%A]") or l:find("favorite") or l:find("favourite")) then
        return true
    end
    if l:find("favorite") or l:find("favourite") then return true end
    return false
end

-- GAME UI INTERACTION
local function isOurGui(instance)
    local node = instance
    for _ = 1, 12 do
        if not node then break end
        if node.Name == UI_NAME then return true end
        node = node.Parent
    end
    return false
end

local function isVisibleChain(inst)
    local node = inst
    while node do
        if node:IsA("GuiObject") and not node.Visible then return false end
        if node:IsA("ScreenGui") then return node.Enabled end
        node = node.Parent
    end
    return true
end

local function findAllTextBoxes(pg)
    local boxes = {}
    for _, gui in ipairs(pg:GetChildren()) do
        if gui:IsA("ScreenGui") and gui.Enabled and not isOurGui(gui) then
            for _, d in ipairs(gui:GetDescendants()) do
                if d:IsA("TextBox") and not isOurGui(d) then boxes[#boxes + 1] = d end
            end
        end
    end
    return boxes
end

local function findCodeButtons(pg)
    local btns = {}
    for _, gui in ipairs(pg:GetChildren()) do
        if gui:IsA("ScreenGui") and gui.Enabled and not isOurGui(gui) then
            for _, d in ipairs(gui:GetDescendants()) do
                if (d:IsA("TextButton") or d:IsA("ImageButton")) and not isOurGui(d) then
                    local n = d.Name:lower()
                    local pn = (d.Parent and d.Parent.Name or ""):lower()
                    if (n:find("code") or n:find("redeem") or pn:find("code") or pn:find("redeem"))
                        and isVisibleChain(d) then
                        btns[#btns + 1] = d
                    end
                end
            end
        end
    end
    return btns
end

local function fireConnections(signal, ...)
    if typeof(getconns) ~= "function" then return false end
    local ok, connections = pcall(getconns, signal)
    if not ok or type(connections) ~= "table" then return false end
    local args = table.pack(...)
    local fired = false
    for _, connection in ipairs(connections) do
        local fireOk = pcall(function()
            if connection.Enabled ~= false then
                if args.n > 0 then connection:Fire(table.unpack(args, 1, args.n))
                else connection:Fire() end
            end
        end)
        fired = fired or fireOk
    end
    return fired
end

local function clickButton(btn)
    if not btn then return false end
    local any = false
    any = pcall(function() btn.MouseButton1Click:Fire() end) or any
    any = pcall(function() btn.Activated:Fire() end) or any
    if typeof(firesignal) == "function" then
        any = pcall(firesignal, btn.MouseButton1Click) or any
        any = pcall(firesignal, btn.Activated) or any
    end
    any = fireConnections(btn.MouseButton1Click) or any
    any = fireConnections(btn.Activated) or any
    if typeof(fireclick) == "function" then any = pcall(fireclick, btn) or any end
    return any
end

local function fireBoxFocusLost(box)
    if not box then return false end
    local any = false
    if typeof(firesignal) == "function" then
        any = pcall(firesignal, box.FocusLost, true) or any
    end
    if typeof(getconns) == "function" then
        local ok, connections = pcall(getconns, box.FocusLost)
        if ok and type(connections) == "table" then
            for _, connection in ipairs(connections) do
                local fn
                pcall(function() fn = connection.Function end)
                if fn and typeof(getupvalues) == "function" and typeof(setupv) == "function" then
                    local upsOk, ups = pcall(getupvalues, fn)
                    if upsOk and type(ups) == "table" then
                        for index, value in pairs(ups) do
                            if type(value) == "boolean" and value == true then
                                pcall(setupv, fn, index, false)
                            end
                        end
                    end
                end
                any = pcall(function()
                    if connection.Enabled ~= false then connection:Fire(true) end
                end) or any
            end
        end
    end
    return any
end

local function releaseFocus(box)
    pcall(function() box:ReleaseFocus(false) end)
    pcall(function()
        local vim = game:GetService("VirtualInputManager")
        if vim then
            vim:SendMouseButtonEvent(0, 0, 0, true, game, 0)
            task.wait(0.03)
            vim:SendMouseButtonEvent(0, 0, 0, false, game, 0)
        end
    end)
    pcall(function() box:ReleaseFocus(false) end)
end

local function currentCodeBox()
    local pg = playerGui or player:FindFirstChildOfClass("PlayerGui")
    if not pg then return nil end
    local codesGui = pg:FindFirstChild("Codes")
    if codesGui then
        local root = codesGui:FindFirstChild("Codes") or codesGui
        local redeem = root:FindFirstChild("CodeRedeem")
        local box = redeem and redeem:FindFirstChild("TextBox")
        if box and box:IsA("TextBox") then return box end
        for _, d in ipairs(codesGui:GetDescendants()) do
            if d:IsA("TextBox") then return d end
        end
    end
    for _, box in ipairs(findAllTextBoxes(pg)) do
        if isVisibleChain(box) then return box end
    end
    return nil
end

local function findSubmitButton(box)
    local names = { "submit", "redeem", "claim", "confirm", "enter", "send", "apply", "ok", "use", "go", "check" }
    local node = box.Parent
    for _ = 1, 8 do
        if not node then break end
        for _, d in ipairs(node:GetDescendants()) do
            if (d:IsA("TextButton") or d:IsA("ImageButton")) and not isOurGui(d) and d ~= box then
                local n = d.Name:lower()
                local txt = ""
                pcall(function() txt = tostring(d.Text):lower() end)
                for _, key in ipairs(names) do
                    if (n:find(key) or txt:find(key)) and isVisibleChain(d) then return d end
                end
            end
        end
        node = node.Parent
    end
    return nil
end

-- types the text into the code box and pushes submit. captureFocus is used
-- for riddle answers so the text sticks even when the box is untouched.
typeAndSubmitCode = function(code, captureFocus)
    code = tostring(code or "")
    if code == "" then return false, "empty code" end

    local pg = playerGui or player:FindFirstChildOfClass("PlayerGui")
    if not pg then return false, "no PlayerGui" end

    -- open the codes menu if it is closed
    local codesGui = pg:FindFirstChild("Codes")
    if codesGui then
        pcall(function()
            if codesGui:IsA("ScreenGui") then codesGui.Enabled = true end
            local node = codesGui:FindFirstChild("Codes") or codesGui
            while node and node ~= codesGui do
                if node:IsA("GuiObject") then node.Visible = true end
                node = node.Parent
            end
        end)
    else
        for _, btn in ipairs(findCodeButtons(pg)) do
            clickButton(btn)
            task.wait(0.05)
        end
        task.wait(0.2)
    end

    local box
    local deadline = tick() + 2
    repeat
        box = currentCodeBox()
        if box then break end
        task.wait(0.1)
    until tick() > deadline
    if not box then return false, "code box not found" end

    if captureFocus then
        pcall(function() box:CaptureFocus() end)
        task.wait(0.05)
    end

    pcall(function() box.Text = "" end)
    task.wait(0.03)
    pcall(function() box.Text = code end)
    for _ = 1, 10 do
        if box.Text == code then break end
        task.wait(0.04)
        pcall(function() box.Text = code end)
    end
    task.wait(0.08)

    local submitted = fireBoxFocusLost(box)

    local btn = findSubmitButton(box)
    if btn then
        clickButton(btn)
        submitted = true
    end

    pcall(function()
        if typeof(keypress) == "function" then
            keypress(0x0D)
            task.wait(0.04)
            if typeof(keyrelease) == "function" then keyrelease(0x0D) end
            submitted = true
        end
    end)

    if captureFocus then
        task.wait(0.05)
        releaseFocus(box)
    end

    return true, submitted and "submitted" or "typed"
end

-- GUI - skyr0.wtf (same palette and primitives as the main hub)
-- PURPLE / WHITE. Purple carries every active state, white carries every
-- readable thing, surfaces are a neutral grey ramp 0C -> 14 -> 1C -> 26 -> 32.
local SKYR_BRAND     = "skyr0"
local SKYR_BRAND_TLD = ".wtf"
local SKYR_TAG       = "skyr0.wtf"
local SKYR_AUTHOR    = "By: skyr0s0"

local Theme = {
    MainBackground = Color3.fromRGB(12, 12, 14),  Background = Color3.fromRGB(20, 20, 23),
    Panel          = Color3.fromRGB(28, 28, 32),  Row        = Color3.fromRGB(38, 38, 43),
    RowHover       = Color3.fromRGB(50, 50, 57),
    Accent         = Color3.fromRGB(167, 110, 247), AccentLight = Color3.fromRGB(139, 84, 222),
    Green          = Color3.fromRGB(167, 110, 247),
    Red            = Color3.fromRGB(226, 72, 80),  Red2 = Color3.fromRGB(190, 54, 62),
    Text           = Color3.fromRGB(255, 255, 255), Dim = Color3.fromRGB(138, 138, 150),
    Stroke         = Color3.fromRGB(44, 44, 50),
    SoftButton     = Color3.fromRGB(38, 38, 43),  SoftButtonHover = Color3.fromRGB(50, 50, 57),
    SoftAccent     = Color3.fromRGB(28, 28, 32),
    ToggleOff      = Color3.fromRGB(58, 58, 66),  ToggleOff2 = Color3.fromRGB(30, 30, 34),
    InputBg        = Color3.fromRGB(16, 16, 19),  SliderBg = Color3.fromRGB(44, 44, 50),
}

-- kept under the old names so the rest of the script needs no changes
local T = {
    bg = Theme.MainBackground, bgDeep = Theme.InputBg, panel = Theme.Panel,
    panel2 = Theme.Row, line = Theme.Stroke, accent = Theme.Accent,
    accent2 = Theme.AccentLight, accentHi = Theme.Accent, text = Theme.Text,
    sub = Theme.Dim, ok = Theme.Green, err = Theme.Red, warn = Color3.fromRGB(250, 204, 90),
}

local LOG = {
    dim   = "rgb(138,138,150)",
    white = "rgb(255,255,255)",
    ok    = "rgb(167,110,247)",
    err   = "rgb(226,72,80)",
    warn  = "rgb(250,204,90)",
    acc   = "rgb(167,110,247)",
    cyan  = "rgb(139,84,222)",
}

-- primitives (identical to the hub's)
local function new(class, props, parent)
    local inst = Instance.new(class)
    for key, value in pairs(props or {}) do inst[key] = value end
    if parent then inst.Parent = parent end
    return inst
end
-- =====================================================================
-- TOUCH / RESPONSIVE METRICS  (matches the hub's rewritten UI core)
-- ---------------------------------------------------------------------
-- MIN_TAP floors every interactive row on a phone -- a 30px button is under
-- the size a finger can reliably hit. The window is clamped to the viewport
-- for the same reason the hub's is: a fixed 300x472 does not fit every phone
-- and there is no way to scroll a window that opens off-screen.
-- =====================================================================
local IS_TOUCH = false
pcall(function()
    IS_TOUCH = UserInputService.TouchEnabled and not UserInputService.MouseEnabled
end)
local MIN_TAP = IS_TOUCH and 40 or 0
local function tapH(h) return math.max(h, MIN_TAP) end
local function viewportSize()
    local cam = workspace.CurrentCamera
    return (cam and cam.ViewportSize) or Vector2.new(1280, 720)
end

local function corner(o, r)
    local c = Instance.new("UICorner"); c.CornerRadius = UDim.new(0, r); c.Parent = o; return c
end
local function stroke(o, col, th, tr)
    local s = Instance.new("UIStroke")
    s.Color = col or Theme.Stroke; s.Thickness = th or 1; s.Transparency = tr or 0
    s.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; s.Parent = o
    return s
end
local function tw(o, p, t)
    TweenService:Create(o, TweenInfo.new(t or 0.14, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), p):Play()
end
local function tween(o, t, p) tw(o, p, t) end
local function addOutline(f)
    local o = Instance.new("UIStroke")
    o.Color = Theme.AccentLight; o.Thickness = 1.25; o.Transparency = 0.08
    o.ApplyStrokeMode = Enum.ApplyStrokeMode.Border; o.Parent = f
    return o
end
-- frosted glass: top-lit depth gradient + a rim light along the top inner edge
local function decoratePanel(f)
    local g = Instance.new("UIGradient", f); g.Rotation = 90
    g.Color = ColorSequence.new{
        ColorSequenceKeypoint.new(0, Color3.fromRGB(255, 255, 255)),
        ColorSequenceKeypoint.new(1, Color3.fromRGB(188, 188, 188)),
    }
    local hl = Instance.new("Frame")
    hl.Size = UDim2.new(1, -16, 0, 1); hl.Position = UDim2.new(0, 8, 0, 1)
    hl.BackgroundColor3 = Color3.fromRGB(243, 243, 243); hl.BackgroundTransparency = 0.5
    hl.BorderSizePixel = 0; hl.ZIndex = 6; hl.Parent = f
    local hlg = Instance.new("UIGradient", hl)
    hlg.Transparency = NumberSequence.new{
        NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(0.5, 0), NumberSequenceKeypoint.new(1, 1),
    }
    return g
end
local function label(parent, text, size, color, font, align)
    return new("TextLabel", {
        BackgroundTransparency = 1, Text = text, TextSize = size or 11,
        TextColor3 = color or Theme.Text, Font = font or Enum.Font.GothamMedium,
        TextXAlignment = align or Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
    }, parent)
end

-- click helper that behaves on both mouse and touch
-- Press fires on RELEASE, and only one path is bound.
--
-- The old version connected Activated AND MouseButton1Click AND a Touch
-- InputBegan, then leaned on a 0.16s debounce to swallow the duplicates that
-- caused -- which also swallowed genuine fast double presses. This binds
-- InputBegan/InputEnded once, so mouse and touch behave identically and
-- dragging off a button cancels it the way a phone user expects.
local function bindClick(button, callback)
    local down = false
    local base = button.BackgroundColor3
    button.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            down = true
            tw(button, { BackgroundTransparency = math.min((button.BackgroundTransparency or 0) + 0.15, 1) }, 0.08)
        end
    end)
    button.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
        or input.UserInputType == Enum.UserInputType.Touch then
            local fire = down
            down = false
            tw(button, { BackgroundColor3 = base }, 0.10)
            if fire then task.spawn(callback) end
        end
    end)
end

-- screen gui + global scale (same wrapper the hub uses)
pcall(function()
    for _, name in ipairs({ UI_NAME, "Skyr0WtfUI", "ACECodeSniperUI", "AutoTypeCodesUI", "ACEPaste", "GuiznxRiddle" }) do
        local old = playerGui:FindFirstChild(name)
        if old then old:Destroy() end
        local oldCore = CoreGui and CoreGui:FindFirstChild(name)
        if oldCore then oldCore:Destroy() end
    end
end)

GUI = new("ScreenGui", {
    Name = UI_NAME,
    ResetOnSpawn = false,
    IgnoreGuiInset = true,
    DisplayOrder = 9999999,
    ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
}, playerGui)

local Master = new("Frame", {
    Name = "Skyr0_MasterFrame",
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    Size = UDim2.new(1, 0, 1, 0),
}, GUI)
local GlobalScale = new("UIScale", { Name = "Skyr0_GlobalScale", Scale = 1 }, Master)

local viewportConn
local function recalculateScale()
    local camera = workspace.CurrentCamera
    if not camera then return end
    local h = camera.ViewportSize.Y
    local scale
    if UserInputService.TouchEnabled then
        scale = math.clamp(h / 1000, 0.52, 0.78)
    else
        scale = math.clamp(h / 800, 0.70, 1.0)
    end
    GlobalScale.Scale = scale
    Master.Size = UDim2.new(1 / scale, 0, 1 / scale, 0)
end
local function setupCameraListener()
    if viewportConn then pcall(function() viewportConn:Disconnect() end) end
    local camera = workspace.CurrentCamera
    if camera then
        viewportConn = camera:GetPropertyChangedSignal("ViewportSize"):Connect(recalculateScale)
    end
    recalculateScale()
end
workspace:GetPropertyChangedSignal("CurrentCamera"):Connect(setupCameraListener)
task.spawn(setupCameraListener)

-- the panel (makeQuickPanel look: "skyr0.wtf" over a subtitle)
-- Fixed on desktop; clamped to the screen on a phone.
local WIN_W, WIN_H = 300, 472
if IS_TOUCH then
    local vp = viewportSize()
    WIN_W = math.floor(math.clamp(vp.X * 0.86, 260, 340))
    WIN_H = math.floor(math.clamp(vp.Y * 0.78, 320, 520))
end

local Window = new("Frame", {
    Name = "CodeRedeemer",
    Size = UDim2.fromOffset(WIN_W, WIN_H),
    Position = UDim2.new(1, -320, 0, 60),
    BackgroundColor3 = Theme.Background,
    BackgroundTransparency = 0.3,
    BorderSizePixel = 0,
    Active = true,
    ClipsDescendants = true,
}, Master)
corner(Window, 12)
local WindowOutline = addOutline(Window)
decoratePanel(Window)

local Header = new("Frame", {
    Name = "Header",
    Size = UDim2.new(1, 0, 0, 48),
    BackgroundTransparency = 1,
    Active = true,
}, Window)

local Brand = label(Header, SKYR_BRAND .. SKYR_BRAND_TLD, 12, Theme.Text, Enum.Font.GothamBlack, Enum.TextXAlignment.Center)
Brand.RichText = true
Brand.Text = SKYR_BRAND .. '<font color="rgb(167,110,247)">' .. SKYR_BRAND_TLD .. "</font>"
Brand.Size = UDim2.new(1, -58, 0, 16)
Brand.Position = UDim2.new(0, 12, 0, 6)

local SubTitle = label(Header, "Code Redeemer", 10, Theme.Dim, Enum.Font.GothamMedium, Enum.TextXAlignment.Center)
SubTitle.Size = UDim2.new(1, -58, 0, 13)
SubTitle.Position = UDim2.new(0, 12, 0, 21)

local LiveDot = new("Frame", {
    Size = UDim2.fromOffset(6, 6),
    Position = UDim2.fromOffset(12, 14),
    BackgroundColor3 = Theme.ToggleOff,
    BorderSizePixel = 0,
}, Header)
corner(LiveDot, 3)

local function headerButton(text, xOffset)
    local b = new("TextButton", {
        Size = UDim2.fromOffset(IS_TOUCH and 26 or 16, IS_TOUCH and 26 or 16),
        Position = UDim2.new(1, IS_TOUCH and (xOffset - 10) or xOffset, 0, IS_TOUCH and 11 or 6),
        BackgroundColor3 = Theme.Row,
        BackgroundTransparency = 0.3,
        BorderSizePixel = 0,
        AutoButtonColor = false,
        Active = true,
        Text = text,
        TextSize = 11,
        TextColor3 = Theme.Text,
        Font = Enum.Font.GothamBold,
    }, Header)
    corner(b, 5)
    stroke(b, Theme.AccentLight, 1, 0.28)
    b.MouseEnter:Connect(function() tw(b, { BackgroundColor3 = Theme.RowHover }, 0.12) end)
    b.MouseLeave:Connect(function() tw(b, { BackgroundColor3 = Theme.Row }, 0.12) end)
    return b
end
local MinBtn   = headerButton("–", -40)
local CloseBtn = headerButton("×", -20)

-- master power pill, same geometry as the hub's row toggles
local Power = new("TextButton", {
    Name = "Power",
    Size = UDim2.fromOffset(38, 21),
    Position = UDim2.new(1, -46, 0, 24),
    BackgroundColor3 = cfg.sniper and Theme.Green or Theme.ToggleOff,
    BorderSizePixel = 0,
    AutoButtonColor = false,
    Active = true,
    Text = "",
}, Header)
corner(Power, 20)
local PowerDot = new("Frame", {
    Size = UDim2.fromOffset(16, 16),
    Position = cfg.sniper and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8),
    BackgroundColor3 = Theme.InputBg,
    BorderSizePixel = 0,
}, Power)
corner(PowerDot, 20)

local HeaderDivider = new("Frame", {
    Size = UDim2.new(1, -24, 0, 1),
    Position = UDim2.new(0, 12, 0, 46),
    BackgroundColor3 = Theme.AccentLight,
    BackgroundTransparency = 0.04,
    BorderSizePixel = 0,
}, Window)
new("UIGradient", {
    Transparency = NumberSequence.new{
        NumberSequenceKeypoint.new(0, 1), NumberSequenceKeypoint.new(0.5, 0), NumberSequenceKeypoint.new(1, 1),
    },
}, HeaderDivider)

-- body
local Body = new("Frame", {
    Name = "Body",
    Size = UDim2.new(1, -12, 1, -100),
    Position = UDim2.fromOffset(6, 52),
    BackgroundTransparency = 1,
}, Window)
new("UIListLayout", {
    FillDirection = Enum.FillDirection.Vertical,
    SortOrder = Enum.SortOrder.LayoutOrder,
    Padding = UDim.new(0, 6),
}, Body)

-- ON/OFF button row, the hub's makeSyncStateRow, used for the master switch
local function makeStateRow(order, text, initial, onChange)
    local row = new("Frame", {
        Size = UDim2.new(1, -4, 0, tapH(34)),
        BackgroundTransparency = 1,
        LayoutOrder = order,
    }, Body)
    local l = label(row, text, 12, Theme.Text, Enum.Font.GothamSemibold)
    l.Size = UDim2.new(1, -84, 1, 0)
    l.Position = UDim2.new(0, 4, 0, 0)
    l.TextTruncate = Enum.TextTruncate.AtEnd

    local btn = new("TextButton", {
        Name = "WhiteTextBtn",
        Size = UDim2.fromOffset(IS_TOUCH and 80 or 72, tapH(30)),
        Position = UDim2.new(1, IS_TOUCH and -82 or -74, 0.5, -tapH(30)/2),
        BackgroundColor3 = initial and Theme.Green or Theme.ToggleOff2,
        BorderSizePixel = 0,
        AutoButtonColor = false,
        Active = true,
        Text = initial and "ON" or "OFF",
        TextColor3 = initial and Color3.fromRGB(15, 15, 15) or Color3.fromRGB(255, 255, 255),
        Font = Enum.Font.GothamBlack,
        TextSize = 12,
    }, row)
    corner(btn, 6)

    local state = initial
    bindClick(btn, function()
        state = not state
        tw(btn, { BackgroundColor3 = state and Theme.Green or Theme.ToggleOff2 }, 0.12)
        btn.Text = state and "ON" or "OFF"
        btn.TextColor3 = state and Color3.fromRGB(15, 15, 15) or Color3.fromRGB(255, 255, 255)
        onChange(state)
    end)
    return btn, function(value)
        state = value
        btn.BackgroundColor3 = value and Theme.Green or Theme.ToggleOff2
        btn.Text = value and "ON" or "OFF"
        btn.TextColor3 = value and Color3.fromRGB(15, 15, 15) or Color3.fromRGB(255, 255, 255)
    end
end

-- pill toggle row, the hub's makeSyncMainToggle
local function makeToggleRow(order, text, initial, onChange)
    local row = new("Frame", {
        Size = UDim2.new(1, -4, 0, tapH(31)),
        BackgroundColor3 = Theme.Panel,
        BackgroundTransparency = 0.3,
        BorderSizePixel = 0,
        LayoutOrder = order,
    }, Body)
    corner(row, 6)

    local l = label(row, text, 10, Theme.Text, Enum.Font.GothamMedium)
    l.Size = UDim2.new(1, -54, 1, 0)
    l.Position = UDim2.new(0, 8, 0, 0)
    l.TextTruncate = Enum.TextTruncate.AtEnd

    local toggle = new("TextButton", {
        Size = UDim2.fromOffset(38, 21),
        Position = UDim2.new(1, -44, 0.5, -10.5),
        BackgroundColor3 = initial and Theme.Green or Theme.ToggleOff,
        BorderSizePixel = 0,
        AutoButtonColor = false,
        Active = true,
        Text = "",
    }, row)
    corner(toggle, 20)

    local dot = new("Frame", {
        Size = UDim2.fromOffset(16, 16),
        Position = initial and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8),
        BackgroundColor3 = Theme.InputBg,
        BorderSizePixel = 0,
    }, toggle)
    corner(dot, 20)

    local state = initial
    bindClick(toggle, function()
        state = not state
        tw(toggle, { BackgroundColor3 = state and Theme.Green or Theme.ToggleOff }, 0.12)
        tw(dot, { Position = state and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8) }, 0.12)
        onChange(state)
    end)
    return toggle
end

local _, setPowerRow = makeStateRow(1, "Code Sniper", cfg.sniper, function(state)
    cfg.sniper = state
    saveConfig()
    applyPowerVisual()
    if state then
        clearCapture()
        logRich('<font color="' .. LOG.ok .. '">sniper on</font>')
    else
        logRich('<font color="' .. LOG.err .. '">sniper off</font>')
    end
end)

makeToggleRow(2, "Auto Submit", cfg.autoSubmit, function(state)
    cfg.autoSubmit = state
    saveConfig()
    logRich('<font color="' .. LOG.dim .. '">auto submit </font><font color="'
        .. (state and LOG.ok or LOG.err) .. '">' .. (state and "on" or "off") .. "</font>")
end)

makeToggleRow(3, "Riddle Solver", cfg.riddleSolver, function(state)
    cfg.riddleSolver = state
    saveConfig()
    logRich('<font color="' .. LOG.dim .. '">riddle solver </font><font color="'
        .. (state and LOG.ok or LOG.err) .. '">' .. (state and "on" or "off") .. "</font>")
end)

makeToggleRow(4, "Retype Invalid", cfg.retypeInvalid, function(state)
    cfg.retypeInvalid = state
    saveConfig()
    logRich('<font color="' .. LOG.dim .. '">retype invalid </font><font color="'
        .. (state and LOG.ok or LOG.err) .. '">' .. (state and "on" or "off") .. "</font>")
end)

-- slider (the hub's makeQuickSlider, snapped to whole parts)
do
    local MIN, MAX = 1, 10
    local holder = new("Frame", {
        Size = UDim2.new(1, -4, 0, 46),
        BackgroundTransparency = 1,
        LayoutOrder = 5,
    }, Body)

    local sliderLabel = label(holder, "Submit after: " .. cfg.submitAfter .. " parts", 10, Theme.Text, Enum.Font.GothamMedium)
    sliderLabel.Size = UDim2.new(1, 0, 0, 16)
    sliderLabel.Position = UDim2.new(0, 4, 0, 0)

    local bar = new("Frame", {
        Size = UDim2.new(1, -10, 0, 6),
        Position = UDim2.fromOffset(4, 26),
        BackgroundColor3 = Theme.SliderBg,
        BorderSizePixel = 0,
    }, holder)
    corner(bar, 10)

    local ratio = (cfg.submitAfter - MIN) / (MAX - MIN)
    local fill = new("Frame", {
        Size = UDim2.new(math.clamp(ratio, 0, 1), 0, 1, 0),
        BackgroundColor3 = Theme.Accent,
        BorderSizePixel = 0,
    }, bar)
    corner(fill, 10)

    local knob = new("Frame", {
        Name = "WhiteSliderKnob",
        Size = UDim2.fromOffset(14, 14),
        AnchorPoint = Vector2.new(0.5, 0.5),
        Position = UDim2.new(math.clamp(ratio, 0, 1), 0, 0.5, 0),
        BackgroundColor3 = Color3.fromRGB(255, 255, 255),
        BorderSizePixel = 0,
    }, bar)
    corner(knob, 20)

    local dragging = false
    local function update(x)
        local rel = math.clamp((x - bar.AbsolutePosition.X) / bar.AbsoluteSize.X, 0, 1)
        local value = math.floor(MIN + (MAX - MIN) * rel + 0.5)
        rel = (value - MIN) / (MAX - MIN)
        fill.Size = UDim2.new(rel, 0, 1, 0)
        knob.Position = UDim2.new(rel, 0, 0.5, 0)
        sliderLabel.Text = "Submit after: " .. value .. (value == 1 and " part" or " parts")
        if value ~= cfg.submitAfter then
            cfg.submitAfter = value
            clearCapture()
            saveConfig()
        end
    end

    bar.InputBegan:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            dragging = true
            update(input.Position.X)
        end
    end)
    UserInputService.InputEnded:Connect(function(input)
        if input.UserInputType == Enum.UserInputType.MouseButton1
            or input.UserInputType == Enum.UserInputType.Touch then
            dragging = false
        end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if dragging and (input.UserInputType == Enum.UserInputType.MouseMovement
            or input.UserInputType == Enum.UserInputType.Touch) then
            update(input.Position.X)
        end
    end)
end

-- console
local ConsoleBar = new("Frame", {
    Size = UDim2.new(1, -4, 0, 16),
    BackgroundTransparency = 1,
    LayoutOrder = 6,
}, Body)

local ConsoleTitle = label(ConsoleBar, "CONSOLE", 9, Theme.Dim, Enum.Font.GothamBlack)
ConsoleTitle.Size = UDim2.fromOffset(60, 16)
ConsoleTitle.Position = UDim2.fromOffset(4, 0)

local ClearBtn = new("TextButton", {
    Size = UDim2.fromOffset(46, 16),
    Position = UDim2.new(1, -46, 0, 0),
    BackgroundColor3 = Theme.Row,
    BackgroundTransparency = 0.3,
    BorderSizePixel = 0,
    AutoButtonColor = false,
    Active = true,
    Text = "CLEAR",
    TextSize = 9,
    TextColor3 = Theme.Text,
    Font = Enum.Font.GothamBold,
}, ConsoleBar)
corner(ClearBtn, 6)
stroke(ClearBtn, Theme.AccentLight, 1, 0.28)
ClearBtn.MouseEnter:Connect(function() tw(ClearBtn, { BackgroundColor3 = Theme.RowHover }, 0.12) end)
ClearBtn.MouseLeave:Connect(function() tw(ClearBtn, { BackgroundColor3 = Theme.Row }, 0.12) end)

local Console = new("ScrollingFrame", {
    Name = "Console",
    Size = UDim2.new(1, -4, 0, 112),
    BackgroundColor3 = Theme.InputBg,
    BackgroundTransparency = 0.15,
    BorderSizePixel = 0,
    Active = true,
    ClipsDescendants = true,
    ScrollingDirection = Enum.ScrollingDirection.Y,
    ScrollBarThickness = 3,
    ScrollBarImageColor3 = Theme.Accent,
    CanvasSize = UDim2.new(0, 0, 0, 0),
    AutomaticCanvasSize = Enum.AutomaticSize.None,
    ElasticBehavior = Enum.ElasticBehavior.WhenScrollable,
    LayoutOrder = 7,
}, Body)
corner(Console, 6)
stroke(Console, Theme.AccentLight, 1, 0.28)

local ConsoleText = new("TextLabel", {
    Name = "Output",
    Size = UDim2.new(1, -14, 0, 0),
    AutomaticSize = Enum.AutomaticSize.Y,
    Position = UDim2.fromOffset(7, 5),
    BackgroundTransparency = 1,
    RichText = true,
    Text = "",
    TextSize = 11,
    Font = Enum.Font.Code,
    TextColor3 = Theme.Text,
    TextXAlignment = Enum.TextXAlignment.Left,
    TextYAlignment = Enum.TextYAlignment.Top,
    TextWrapped = true,
}, Console)

-- manual riddle tester
local AskRow = new("Frame", {
    Size = UDim2.new(1, -4, 0, 26),
    BackgroundTransparency = 1,
    LayoutOrder = 8,
}, Body)

local AskBox = new("TextBox", {
    Size = UDim2.new(1, -62, 1, 0),
    BackgroundColor3 = Theme.InputBg,
    BackgroundTransparency = 0.15,
    BorderSizePixel = 0,
    ClearTextOnFocus = false,
    Text = "",
    PlaceholderText = "test a riddle...",
    PlaceholderColor3 = Theme.Dim,
    TextSize = 10,
    Font = Enum.Font.GothamMedium,
    TextColor3 = Theme.Text,
    TextXAlignment = Enum.TextXAlignment.Left,
}, AskRow)
corner(AskBox, 6)
stroke(AskBox, Theme.AccentLight, 1, 0.28)
new("UIPadding", { PaddingLeft = UDim.new(0, 8), PaddingRight = UDim.new(0, 6) }, AskBox)

local AskBtn = new("TextButton", {
    Size = UDim2.fromOffset(58, 26),
    Position = UDim2.new(1, -58, 0, 0),
    BackgroundColor3 = Theme.Green,
    BorderSizePixel = 0,
    AutoButtonColor = false,
    Active = true,
    Text = "SOLVE",
    TextSize = 11,
    TextColor3 = Color3.fromRGB(15, 15, 15),
    Font = Enum.Font.GothamBlack,
}, AskRow)
corner(AskBtn, 6)
AskBtn.MouseEnter:Connect(function() tw(AskBtn, { BackgroundColor3 = Theme.AccentLight }, 0.12) end)
AskBtn.MouseLeave:Connect(function() tw(AskBtn, { BackgroundColor3 = Theme.Green }, 0.12) end)

-- bottom bar (mirrors the hub's)
local BottomBar = new("Frame", {
    Name = "BottomBar",
    Size = UDim2.new(1, -12, 0, 38),
    Position = UDim2.new(0, 6, 1, -44),
    BackgroundColor3 = Theme.Background,
    BackgroundTransparency = 0.3,
    BorderSizePixel = 0,
}, Window)
corner(BottomBar, 10)
stroke(BottomBar, Theme.AccentLight, 1, 0.28)

local LogoTile = new("Frame", {
    Size = UDim2.fromOffset(26, 26),
    Position = UDim2.new(0, 8, 0.5, -13),
    BackgroundColor3 = Theme.SoftAccent,
    BorderSizePixel = 0,
}, BottomBar)
corner(LogoTile, 8)
stroke(LogoTile, Theme.AccentLight, 1, 0.4)

local WordMark = label(BottomBar, SKYR_BRAND, 15, Theme.AccentLight, Enum.Font.GothamBlack)
WordMark.Size = UDim2.fromOffset(46, 20)
WordMark.Position = UDim2.fromOffset(40, 4)

local BarDivider = label(BottomBar, "|", 14, Theme.AccentLight, Enum.Font.GothamBlack)
BarDivider.Size = UDim2.fromOffset(10, 20)
BarDivider.Position = UDim2.fromOffset(84, 4)

local FullMark = label(BottomBar, SKYR_TAG, 12, Theme.AccentLight, Enum.Font.GothamBold)
FullMark.Size = UDim2.fromOffset(80, 20)
FullMark.Position = UDim2.fromOffset(96, 4)

local Author = label(BottomBar, SKYR_AUTHOR, 8, Theme.Dim, Enum.Font.GothamSemibold)
Author.Size = UDim2.fromOffset(120, 12)
Author.Position = UDim2.fromOffset(41, 22)

local SolvedLabel = label(BottomBar, "0 solved / 0 asked", 9, Theme.Green, Enum.Font.GothamBold, Enum.TextXAlignment.Right)
SolvedLabel.Size = UDim2.fromOffset(110, 38)
SolvedLabel.Position = UDim2.new(1, -118, 0, 0)

-- console writers
local logLines = {}
local MAX_LOG = 160

local function scrollToBottom()
    task.defer(function()
        task.wait()
        if not Console or not ConsoleText then return end
        local height = ConsoleText.AbsoluteSize.Y + 12
        Console.CanvasSize = UDim2.new(0, 0, 0, height)
        Console.CanvasPosition = Vector2.new(0, math.max(0, height - Console.AbsoluteSize.Y))
    end)
end
ConsoleText:GetPropertyChangedSignal("AbsoluteSize"):Connect(function()
    Console.CanvasSize = UDim2.new(0, 0, 0, ConsoleText.AbsoluteSize.Y + 12)
end)

logRich = function(line)
    logLines[#logLines + 1] = line
    if #logLines > MAX_LOG then table.remove(logLines, 1) end
    ConsoleText.Text = table.concat(logLines, "\n")
    scrollToBottom()
end

clearLog = function()
    logLines = {}
    ConsoleText.Text = '<font color="' .. LOG.dim .. '">cleared</font>'
    Console.CanvasPosition = Vector2.new(0, 0)
end

setStatus = function(msg, color)
    logRich('<font color="' .. (color or LOG.dim) .. '">' .. tostring(msg) .. "</font>")
end

bumpSolvedLabel = function()
    SolvedLabel.Text = _solvedCount .. " solved / " .. _askedCount .. " asked"
end

bindClick(ClearBtn, function()
    clearLog()
    _solvedCount, _askedCount = 0, 0
    bumpSolvedLabel()
end)

-- power visual + header controls
applyPowerVisual = function()
    local on = cfg.sniper
    tw(Power, { BackgroundColor3 = on and Theme.Green or Theme.ToggleOff }, 0.12)
    tw(PowerDot, { Position = on and UDim2.new(1, -19, 0.5, -8) or UDim2.new(0, 3, 0.5, -8) }, 0.12)
    tw(LiveDot, { BackgroundColor3 = on and Theme.Green or Theme.ToggleOff }, 0.12)
    SubTitle.Text = on and "Code Redeemer — listening" or "Code Redeemer — paused"
    SubTitle.TextColor3 = on and Theme.Accent or Theme.Dim
    tw(WindowOutline, { Transparency = on and 0.08 or 0.45 }, 0.12)
    if setPowerRow then setPowerRow(on) end
end

bindClick(Power, function()
    cfg.sniper = not cfg.sniper
    saveConfig()
    applyPowerVisual()
    if cfg.sniper then
        clearCapture()
        logRich('<font color="' .. LOG.ok .. '">sniper on</font>')
    else
        logRich('<font color="' .. LOG.err .. '">sniper off</font>')
    end
end)

local minimised = false
bindClick(MinBtn, function()
    minimised = not minimised
    Body.Visible = not minimised
    BottomBar.Visible = not minimised
    tw(Window, { Size = UDim2.fromOffset(WIN_W, minimised and 50 or WIN_H) }, 0.18)
    MinBtn.Text = minimised and "+" or "–"
end)

bindClick(CloseBtn, function()
    if env.Skyr0Stop then env.Skyr0Stop() else GUI:Destroy() end
end)

-- =====================================================================
-- MOBILE LAUNCHER
-- ---------------------------------------------------------------------
-- Same round button the hub gained. On a phone there is no keyboard and no
-- close/reopen path once the window is hidden, so without this the script is
-- unrecoverable after the first close. Drag it anywhere; a tap toggles.
-- =====================================================================
if IS_TOUCH then
    local fab = new("TextButton", {
        Name = "Skyr0_AceLauncher",
        Size = UDim2.fromOffset(52, 52),
        Position = UDim2.new(0, 14, 0.5, -26),
        BackgroundColor3 = Theme.Accent,
        BorderSizePixel = 0,
        AutoButtonColor = false,
        Active = true,
        Text = "ace",
        TextSize = 13,
        TextColor3 = Color3.fromRGB(20, 20, 20),
        Font = Enum.Font.GothamBlack,
        ZIndex = 50,
    }, Master)
    corner(fab, 26); stroke(fab, Theme.Text, 1.5, 0.6)

    -- drag vs tap: only a finger that barely moved counts as a tap
    local dragging, moved, startIn, startPos = false, false, nil, nil
    fab.InputBegan:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.Touch
        or i.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging, moved = true, false
            startIn, startPos = i.Position, fab.Position
            tw(fab, { Size = UDim2.fromOffset(48, 48) }, 0.08)
        end
    end)
    fab.InputEnded:Connect(function(i)
        if i.UserInputType == Enum.UserInputType.Touch
        or i.UserInputType == Enum.UserInputType.MouseButton1 then
            dragging = false
            tw(fab, { Size = UDim2.fromOffset(52, 52) }, 0.10)
            if not moved then Window.Visible = not Window.Visible end
        end
    end)
    UserInputService.InputChanged:Connect(function(i)
        if not dragging then return end
        if i.UserInputType ~= Enum.UserInputType.Touch
        and i.UserInputType ~= Enum.UserInputType.MouseMovement then return end
        local d = i.Position - startIn
        if math.abs(d.X) > 6 or math.abs(d.Y) > 6 then moved = true end
        local vp = viewportSize()
        fab.Position = UDim2.new(
            0, math.clamp(startPos.X.Offset + d.X, 0, vp.X - 52),
            0, math.clamp(startPos.Y.Offset + d.Y, 0, vp.Y - 52))
    end)
end

-- dragging (scale aware, like the hub's makeDraggable)
do
    local dragging, dragInput, dragStart, startPos = false, nil, nil, nil
    local THRESHOLD = UserInputService.TouchEnabled and 8 or 2
    local moved = false

    local function overControl(position)
        for _, control in ipairs({ Power, MinBtn, CloseBtn }) do
            local pos, size = control.AbsolutePosition, control.AbsoluteSize
            if position.X >= pos.X - 8 and position.X <= pos.X + size.X + 8
                and position.Y >= pos.Y - 8 and position.Y <= pos.Y + size.Y + 8 then
                return true
            end
        end
        return false
    end

    Header.InputBegan:Connect(function(input)
        if input.UserInputType ~= Enum.UserInputType.MouseButton1
            and input.UserInputType ~= Enum.UserInputType.Touch then return end
        if dragging or overControl(input.Position) then return end
        dragging, dragInput = true, input
        dragStart = Vector2.new(input.Position.X, input.Position.Y)
        startPos = Window.Position
        moved = false
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End
                or input.UserInputState == Enum.UserInputState.Cancel then
                if input == dragInput then dragging, dragInput = false, nil end
            end
        end)
    end)

    UserInputService.InputChanged:Connect(function(input)
        if not dragging or not dragInput then return end
        local trackedTouch = dragInput.UserInputType == Enum.UserInputType.Touch and input == dragInput
        local trackedMouse = dragInput.UserInputType == Enum.UserInputType.MouseButton1
            and input.UserInputType == Enum.UserInputType.MouseMovement
        if not trackedTouch and not trackedMouse then return end

        local delta = Vector2.new(input.Position.X, input.Position.Y) - dragStart
        if not moved then
            if delta.Magnitude < THRESHOLD then return end
            moved = true
        end
        local scale = GlobalScale.Scale
        if scale <= 0 then scale = 1 end
        Window.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + (delta.X / scale),
            startPos.Y.Scale, startPos.Y.Offset + (delta.Y / scale)
        )
    end)
end

-- hide / show with RightControl
UserInputService.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.RightControl then
        GUI.Enabled = not GUI.Enabled
    end
end)

-- open animation, same as the hub's openAnim
do
    local target = Window.Position
    local openScale = new("UIScale", { Name = "Skyr0Scale", Scale = 0.92 }, Window)
    Window.Position = UDim2.new(target.X.Scale, target.X.Offset, target.Y.Scale, target.Y.Offset + 18)
    tw(openScale, { Scale = 1 }, 0.20)
    tw(Window, { Position = target }, 0.20)
end

applyPowerVisual()
bumpSolvedLabel()
logRich('<font color="' .. LOG.acc .. '">skyr0.wtf loaded</font>')
logRich('<font color="' .. LOG.dim .. '">example: what is my name and what is my favorite food?</font>')

-- CODE CAPTURE + SUBMIT FLOW
clearCapture = function() _capturedParts = {} end

local function clearBoxWatchers()
    if _boxTextConn then pcall(function() _boxTextConn:Disconnect() end) end
    if _boxAncestryConn then pcall(function() _boxAncestryConn:Disconnect() end) end
    _boxTextConn, _boxAncestryConn, _lastWatchedBox = nil, nil, nil
end

local function watchBox(box)
    if not box or _lastWatchedBox == box then return end
    clearBoxWatchers()
    _lastWatchedBox = box
    if box.Text ~= "" then _lastNonBlankText = box.Text end
    _boxTextConn = box:GetPropertyChangedSignal("Text"):Connect(function()
        if box.Text == "" then clearCapture() else _lastNonBlankText = box.Text end
    end)
    _boxAncestryConn = box.AncestryChanged:Connect(function(_, parent)
        if not parent then clearCapture() clearBoxWatchers() end
    end)
end

clearPending = function()
    _pendingToken += 1
    _pendingText, _pendingBox, _pendingUntil = nil, nil, 0
end

rememberPending = function(box, text, replaceExisting)
    if not cfg.retypeInvalid or not text or text == "" then return end
    if not replaceExisting and _pendingText and os.clock() <= _pendingUntil then return end
    _pendingToken += 1
    local token = _pendingToken
    _pendingText, _pendingBox = text, box
    _pendingUntil = os.clock() + 8
    task.delay(8, function()
        if token == _pendingToken then clearPending() end
    end)
end

local function restoreRejected(box, text)
    if not cfg.retypeInvalid or not text or text == "" then return false end
    RunService.Heartbeat:Wait()
    local target = currentCodeBox() or box
    if not target or not isVisibleChain(target) then return false end
    local ok = pcall(function() target.Text = text end)
    if ok then
        _lastBox = target
        watchBox(target)
    end
    return ok
end

handleFeedback = function(text, sourceObject)
    if not cfg.retypeInvalid or not _pendingText then return end
    if os.clock() > _pendingUntil then clearPending() return end
    if sourceObject and sourceObject:IsDescendantOf(GUI) then return end
    local lower = tostring(text or ""):lower()
    local rejected = lower:find("invalid code", 1, true)
        or lower:find("code is invalid", 1, true)
        or lower:find("expired", 1, true)
        or lower:find("already redeemed", 1, true)
        or lower:find("already used", 1, true)
        or lower:find("doesn't exist", 1, true)
        or lower:find("does not exist", 1, true)
        or lower:find("not found", 1, true)
        or lower:find("rejected", 1, true)
    if not rejected then return end

    local previousText, previousBox = _pendingText, _pendingBox
    local restored = restoreRejected(previousBox, previousText)
    clearPending()
    if restored then
        setStatus("invalid, retyped: " .. previousText, LOG.warn)
    end
end

appendToBox = function(text)
    if not text or text == "" then return end
    if _lastWatchedBox and not isVisibleChain(_lastWatchedBox) then
        clearCapture()
        clearBoxWatchers()
    end

    local box = currentCodeBox()
    _capturedParts[#_capturedParts + 1] = text
    local combined = table.concat(_capturedParts)
    local count = #_capturedParts

    if box then
        _lastBox = box
        watchBox(box)
        local wasFocused = UserInputService:GetFocusedTextBox() == box
        pcall(function() box.Text = combined end)
        if wasFocused then
            pcall(function()
                local caret = #combined + 1
                box.CursorPosition = caret
                box.SelectionStart = caret
            end)
        end
    end

    logRich('<font color="' .. LOG.dim .. '">code ' .. count .. "/" .. cfg.submitAfter
        .. ': </font><font color="' .. LOG.ok .. '">' .. combined .. "</font>")

    if count >= cfg.submitAfter then
        _capturedParts = {}
        if cfg.autoSubmit then
            rememberPending(box, combined, true)
            local ok, message = typeAndSubmitCode(combined, false)
            if ok then
                setStatus("redeemed: " .. combined, LOG.ok)
            else
                local restored = restoreRejected(box, combined)
                clearPending()
                if restored then
                    setStatus("invalid, retyped: " .. combined, LOG.warn)
                else
                    setStatus("failed: " .. tostring(message), LOG.err)
                end
            end
        end
    end
end

-- RIDDLE QUEUE
local function runRiddle(question)
    _askedCount += 1
    bumpSolvedLabel()

    logRich('<font color="' .. LOG.dim .. '">riddle: </font>'
        .. '<font color="' .. LOG.white .. '">' .. question:sub(1, 90) .. "</font>")

    local answer, breakdown, note = solveRiddle(question)

    if breakdown then
        for _, line in ipairs(breakdown) do
            local colour = line:sub(-1) == "?" and LOG.err or LOG.dim
            logRich('<font color="' .. colour .. '">  ' .. line .. "</font>")
        end
    end

    if not answer or answer == "" then
        logRich('<font color="' .. LOG.err .. '">  ' .. tostring(note or "not in database") .. "</font>")
        return
    end

    _solvedCount += 1
    bumpSolvedLabel()
    logRich('<font color="' .. LOG.dim .. '">  answer: </font><font color="' .. LOG.ok .. '">' .. answer .. "</font>")
    if note then
        logRich('<font color="' .. LOG.warn .. '">  ' .. note .. "</font>")
    end

    if cfg.autoSubmit then
        local ok, message = typeAndSubmitCode(answer, true)
        logRich('<font color="' .. LOG.dim .. '">  ' .. (ok and message or ("submit failed: " .. tostring(message))) .. "</font>")
    else
        logRich('<font color="' .. LOG.dim .. '">  auto submit is off, not sent</font>')
    end
end

local function pumpQueue()
    if _riddleBusy then return end
    _riddleBusy = true
    task.spawn(function()
        while #_riddleQueue > 0 do
            local question = table.remove(_riddleQueue, 1)
            pcall(runRiddle, question)
        end
        _riddleBusy = false
    end)
end

local function queueRiddle(question)
    if #_riddleQueue > 6 then return end
    _riddleQueue[#_riddleQueue + 1] = question
    pumpQueue()
end

bindClick(AskBtn, function()
    local question = trim(AskBox.Text)
    if question == "" then
        setStatus("type a riddle first", LOG.warn)
        return
    end
    AskBox.Text = ""
    queueRiddle(question)
end)
AskBox.FocusLost:Connect(function(enterPressed)
    if not enterPressed then return end
    local question = trim(AskBox.Text)
    if question == "" then return end
    AskBox.Text = ""
    queueRiddle(question)
end)

-- ANNOUNCEMENT LISTENER
local function tokenize(text)
    local words = {}
    for word in text:gmatch("[%w_]+") do words[#words + 1] = word end
    return words
end

local _lastRiddleKey, _lastRiddleAt = "", 0

local function onAnnouncement(...)
    local text = trim(stripRich(tostring((...) or "")))
    if text == "" then return end

    if text:find("%s") then
        if not (cfg.riddleSolver and looksLikeRiddle(text)) then return end
        -- the same riddle can be broadcast twice; don't answer it twice
        local key = basicClean(text)
        if key == _lastRiddleKey and tick() - _lastRiddleAt < 5 then return end
        _lastRiddleKey, _lastRiddleAt = key, tick()
        queueRiddle(text)
        return
    end

    -- single word -> code fragment
    for _, word in ipairs(tokenize(text)) do
        if word ~= "" and not _seen[word] then
            _seen[word] = true
            task.delay(1.25, function() _seen[word] = nil end)
            appendToBox(word)
        end
    end
end

-- two independent ways of finding the notification remote
local function resolveNotifyRemote()
    if _G.PhiNotifyRemote then return _G.PhiNotifyRemote end

    -- 1) require the NotificationController and read its upvalues
    local ok, controller = pcall(function()
        if not ReplicatedStorage then return nil end
        local controllers = ReplicatedStorage:FindFirstChild("Controllers")
        local notification = controllers and controllers:FindFirstChild("NotificationController", true)
        if notification then return require(notification) end
        return nil
    end)
    if ok and type(controller) == "table" and type(controller.Start) == "function"
        and typeof(getupvalues) == "function" then
        local valuesOk, values = pcall(getupvalues, controller.Start)
        if valuesOk and type(values) == "table" then
            for _, value in pairs(values) do
                if typeof(value) == "Instance"
                    and (value:IsA("RemoteEvent") or value:IsA("UnreliableRemoteEvent")) then
                    return value
                end
            end
        end
    end

    -- 2) scan Net remotes for a connection owned by NotificationController
    local getinfo = debug and (debug.getinfo or debug.info)
    local packages = ReplicatedStorage and ReplicatedStorage:FindFirstChild("Packages")
    local net = packages and packages:FindFirstChild("Net")
    if net and getinfo and getconns then
        for _, d in ipairs(net:GetDescendants()) do
            if d:IsA("RemoteEvent") then
                local okConns, connections = pcall(getconns, d.OnClientEvent)
                if okConns and type(connections) == "table" then
                    for _, connection in ipairs(connections) do
                        local fnOk, fn = pcall(function() return connection.Function end)
                        if fnOk and type(fn) == "function" then
                            local infoOk, info = pcall(getinfo, fn)
                            if infoOk and info
                                and tostring(info.short_src or info.source or ""):find("NotificationController", 1, true) then
                                return d
                            end
                        end
                    end
                end
            end
        end
    end
    return nil
end

local listenConn
task.spawn(function()
    local remote = resolveNotifyRemote()
    if remote and (remote:IsA("RemoteEvent") or remote:IsA("UnreliableRemoteEvent")) then
        listenConn = remote.OnClientEvent:Connect(function(...)
            if not cfg.sniper then return end
            pcall(onAnnouncement, ...)
        end)
        logRich('<font color="' .. LOG.dim .. '">listener attached: ' .. remote.Name .. "</font>")
    else
        LiveDot.BackgroundColor3 = T.err
        logRich('<font color="' .. LOG.err .. '">no notify remote, use the box below</font>')
    end
end)

-- FEEDBACK WATCHERS + FOCUS TRACKING
local function watchFeedbackObject(obj)
    if not (obj:IsA("TextLabel") or obj:IsA("TextButton")) then return end
    if isOurGui(obj) then return end
    handleFeedback(obj.Text or "", obj)
    obj:GetPropertyChangedSignal("Text"):Connect(function()
        handleFeedback(obj.Text or "", obj)
    end)
end

for _, obj in ipairs(playerGui:GetDescendants()) do pcall(watchFeedbackObject, obj) end
playerGui.DescendantAdded:Connect(function(obj)
    task.wait(0.04)
    pcall(watchFeedbackObject, obj)
end)

UserInputService.TextBoxFocused:Connect(function(box)
    if box:IsDescendantOf(GUI) then return end
    if box ~= currentCodeBox() then return end
    _focused, _lastBox = box, box
    watchBox(box)
end)

UserInputService.TextBoxFocusReleased:Connect(function(box)
    if box:IsDescendantOf(GUI) then return end
    local codeBox = currentCodeBox()
    if box ~= codeBox and box ~= _lastBox then return end
    if cfg.retypeInvalid then
        local submitted = box.Text ~= "" and box.Text or _lastNonBlankText
        rememberPending(box, submitted, false)
    end
    if _focused == box then _focused = nil end
end)

-- CLEANUP HOOK
local function stopEverything()
    if listenConn then
        pcall(function() listenConn:Disconnect() end)
        listenConn = nil
    end
    if viewportConn then pcall(function() viewportConn:Disconnect() end) end
    clearBoxWatchers()
    if GUI then pcall(function() GUI:Destroy() end) end
end

env.Skyr0Stop = stopEverything
env.StopAura = stopEverything

print("skyr0 redeemer source by solar")
