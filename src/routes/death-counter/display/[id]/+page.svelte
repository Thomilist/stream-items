<script lang="ts">
    import type { Member } from "@prisma/client";
    import type { PageData } from "./$types";
    import { onMount } from "svelte";
    import { DEATH_COUNTER_EVENT } from "$lib/death-counter/named";

    export let data: PageData;

    let death_counter = data.death_counter;

    $: show_sum = death_counter.sumLabel ? true : false;

    function reducer(accumulator: number, current_value: Member): number
    {
        return accumulator + current_value.deaths;
    }

    onMount(() =>
    {
        const sse = new EventSource(`/death-counter/display/${data.death_counter.id}`);

        sse.addEventListener(DEATH_COUNTER_EVENT.update, (event) =>
        {
            let received_data = JSON.parse(event.data);

            if (received_data.id === death_counter.id)
            {
                received_data.members.sort((a: Member, b: Member) =>
                {
                    return a.sortIndex - b.sortIndex
                });

                death_counter = received_data;
            }
        });

        return () => sse.close();
    });
</script>

<style>
    @import "style.css";
</style>

<svelte:head>
    <title>{death_counter.name}</title>
</svelte:head>

<table>
    {#each death_counter.members as member}
        <tr>
            <td class="member_name">
                {member.name}:
            </td>
            <td class="member_deaths">
                {member.deaths}
            </td>
        </tr>
    {/each}

    {#if show_sum}
        <tr>
            <td class="sum_label">
                {death_counter.sumLabel}:
            </td>
            <td class="sum">
                {death_counter.members.reduce(reducer, 0)}
            </td>
        </tr>
    {/if}
</table>