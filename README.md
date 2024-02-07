function tablelength(T)
        local hash = {}
        local res = {}

        for _,v in ipairs(T) do
           if (not hash[v]) then
               res[#res+1] = v
               hash[v] = true
           end
        end

        local count = 0
        for _ in pairs(res) do count = count + 1 end
        return count
end

function getNbConfPart() 
    local tab={};

    --conferences count (without eliminating duplicates )
    for key, value in pairs(prosody.full_sessions) do
            if tostring(value["username"]:lower()) ~= "focus" then
                    table.insert(tab,tostring(value["jitsi_meet_room"]:lower()));
            end
    end

    local nbPart = 0
    if it.count(it.keys(prosody.full_sessions))-1 >= 0 --participants count
            then nbPart= it.count(it.keys(prosody.full_sessions))-1
    end

    local nbConf = tablelength(tab) --conferences count
    if nbPart == 0
            then nbConf=0
    end

     --create result as json
    local result_json=array();
    result_json:push({
                    part= nbPart,
                    conf=tablelength(tab) -- eliminates count number of rooms
            });

    -- create json response 
    return json.encode(result_json);
end

function module.load()
    module:depends("http");
	module:provides("http", {
		default_path = "/";
		route = {
			["GET room-size"] = function (event) return wrap_async_run(event,handle_get_room_size) end;
			["GET sessions"] = function () return tostring(it.count(it.keys(prosody.full_sessions))); end;
			["GET room"] = function (event) return wrap_async_run(event,handle_get_room) end;
            ["GET nbConfPart"] = function (event) return wrap_async_run(event,getNbConfPart) end;
		};
	});
end
