# `regen.xml`

This file controls functions that are used for *scripted* regen. (Normal stat regen is set on a per stat case in [stats.xml](stats.xml.md).

The only attribute that is needed is the `script` attribute, which determines which script functions should be called for regen.

~~~xml
<?xml version='1.0' encoding='UTF-8'?>
<chameleon_data>
  <regens>
    <regen script="somefun"/>
    <regen script="longregen"/>
  </regens>
</chameleon_data>
~~~

Both the `somefun` and `longregen` scripts will be called when a player should have a regen. If the player has been unloaded from server memory (due to inactivity or server restart for example) then the function may be called later than expected and the scripts are expected to handle this case correctly.

Here's an example lua function that does "the right thing".

~~~lua
function somefun( p, expected_time, actual_time )
    local dt = 10

    -- We get a nil or 0 on first install as expected_time, and
    -- return the next time it needs to get run.
    if( expected_time==nil or expected_time==0 ) then
       return actual_time + dt
    end

    -- Calculate how many times the regen should have occurred,
    -- incase we have been called later than expected.
    local ncount = 1 + math.floor( ( actual_time - expected_time ) / dt )

    -- Perform the regen
    player.increment_stat(p, "health", ncount )

    -- Return when we next expect to run.
    -- If we returned nil here we would not be called until next
    -- player reload.

    -- Note that we use expected_time rather than actual_time as
    -- the base for the next calculation. This prevents timer drift.
    return expected_time + dt * ncount
end
~~~
