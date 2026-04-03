-- ly
local cas = game:GetService("ContextActionService")
local plrs = game:GetService("Players")
local lplr = plrs.LocalPlayer

local casPlus = {}

function casPlus:GetTouchGui(wait:number):ScreenGui
return lplr.PlayerGui[wait and "WaitForChild" or "FindFirstChild"](
lplr.PlayerGui, "TouchGui", type(wait) == "number" and wait or (wait and math.huge or nil)
)
end

function casPlus:GetActionGui(wait:boolean):ScreenGui
return lplr.PlayerGui[wait and "WaitForChild" or "FindFirstChild"](
lplr.PlayerGui, "ContextActionGui", type(wait) == "number" and wait or (wait and math.huge or nil)
)
end

function casPlus:BindAction(actionName:string, functionToBind:any, createTouchButton:any, ...:KeyCodes):void
local isFunction = type(createTouchButton) == "function"

self.lastActionName, self.lastOnCreateFunction = actionName, isFunction and createTouchButton or nil  
    cas:BindAction(actionName, functionToBind, createTouchButton and true or false, ...)  
          
        if (isFunction and not self.touchButtonConn) then  
            local gui = self:GetActionGui()  
            local frame = gui and gui:FindFirstChild("ContextButtonFrame")  
            if (not frame) then return end  
              
            local button = frame:GetChildren()[1]  
            button.Name = actionName  
              
            local s, w = pcall(createTouchButton, actionName, button)  
            if (not s) then  
                warn(s)  
            end  
              
            self.touchButtonConn = frame.ChildAdded:Connect(function(button)  
                if (  
                    not button:IsA("ImageButton") or button.Name ~= "ContextActionButton" or  
                    type(self.lastActionName) ~= "string" or type(self.lastOnCreateFunction) ~= "function"  
                        )  
                        then return end  
                          
                        local lastActionName, lastOnCreateFunction = self.lastActionName, self.lastOnCreateFunction  
                        self.lastActionName, self.lastOnCreateFunction = nil, nil  
                        task.wait()  
                          
                        button.Name = lastActionName  
                        lastOnCreateFunction(button.Name, button)  
                    end)  
                end  
            end  
              
            local function genFunction(t, name)  
                rawset(t, name, function(_, ...)  
                    cas[name](cas, ...)  
                end)  
                return rawget(t, name)  
            end  
              
            local unsafe = {}  
            local function check(i)  
                if (table.find(unsafe, i)) then return end  
                local s, r = pcall(function()return type(cas[i]) == "function"end)  
                    if (s) then return r else table.insert(unsafe, i) end  
                end  
                  
                setmetatable(casPlus, {  
                __index = function(t, i)  
                    return check(i) and genFunction(t, i) or (not table.find(unsafe, i) and cas[i])  
                end,  
                __newindex = function(t, i, v)  
                    rawset(t, i, v)  
                    return check(i) and genFunction(t, i)  
                end  
                })  
                  
                casPlus:BindAction("CASPLUSTEMP", function() end, true, Enum.KeyCode.Unknown)  
                    casPlus:UnbindAction("CASPLUSTEMP")  
                      
                    return casPlus
