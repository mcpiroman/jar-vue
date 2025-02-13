<template>
    <span v-if="replayEvent.type == ReplayEventType.Message">
        <span class="time">{{ timeString }}</span>
        <span v-if="isContinuationMessage">
            <span class="continue">
                <i class="fa fa-angle-right"></i>
            </span>
        </span>
        <span v-else>
            <span class="modes">
                <div v-if="replayEvent.userSubscriptionMonths > 0 && jadiscoBadgesInfo != null" class="mode">
                    <a href="https://www.poorchat.net/subscriptions/jadisco" target="_blank">
                        <img :src="`https://static.poorchat.net/badges/${getSubscriptionBadgedName(replayEvent.userSubscriptionMonths)}/1x`">
                    </a>
                </div>
                <div v-for="mode in replayEvent.userModes" v-bind:key="mode" class="mode">
                    <img :src="`https://static.poorchat.net/badges/${USER_MODES_TO_LETTER_MAPPING.get(mode)}/1x`">
                </div>
            </span>
            <span class="user" :style="{color: replayEvent.userColor || getRandomUserColor(replayEvent.user)}">{{replayEvent.user}}</span>
            <span>:</span>
        </span>
        <span class="text">
            <span>
                <template v-for="(frag, index) in insertEmoticons(replayEvent.message)">
                    <template v-if="typeof frag == 'string'">
                        {{ frag }}
                    </template>
                    <template v-else>
                        <img class="emoticon" :src="`https://static.poorchat.net/emoticons/${frag.fileName}/1x`" :alt="frag.name" :tooltip="frag.name" v-bind:key="index"/>
                    </template>
                </template>
            </span>
        </span>
        <div v-if="replayEvent.msgid">
            <app-chat-embed v-for="embedEvent in embedsMap.get(replayEvent.msgid)" 
                :key="embedEvent.eventUid" 
                :embedData="embedEvent.embedData">
            </app-chat-embed>
        </div>
    </span>
</template>
<script lang="ts">
import Vue from "vue"
import Chance from 'chance'
import { Component, Prop } from 'vue-property-decorator'
import { UserMode, ReplayEvent, ReplayEventType, ReplayEmbedEvent, EmoticonViewData } from "../../helpers/chatReplay"
//@ts-ignore
import AppChatEmbed from './ChatEmbed'

@Component({
    components: { AppChatEmbed },
    name: 'ChatEvent'
})
export default class extends Vue {
    readonly USER_MODES_TO_LETTER_MAPPING = new Map<UserMode, string>([
        ['admin', 'a'],
        ['globalModerator', 'm'],
        ['owner', 'q'],
        ['moderator', 'o'],
        ['vip', 'h'],
    ])
    readonly USER_COLORS = ["#FF0000", "#FF8000", "#FFFF00", "#80FF00", "#008000", "#00FF80", "#00FFFF", "#0080FF", "#0000FF", "#8000FF", "#FF00FF", "#FF0080"]
    readonly ReplayEventType = ReplayEventType
    
    @Prop() replayEvent: ReplayEvent
    @Prop() embedsMap: Map<string, ReplayEmbedEvent[]>
    @Prop() isContinuationMessage: boolean
    @Prop() streamStartTime: Date
    @Prop() emoticonsInfo: any
    @Prop() jadiscoBadgesInfo: any
    
    private insertEmoticons(msg): (string | EmoticonViewData)[] | null{
        /*  if(this.replayEvent.type != ReplayEventType.Message)
                return null */
            
        let fragments: (string | EmoticonViewData)[] = [msg]
        
        if(this.emoticonsInfo == null)
            return fragments
        
        for(const emo of this.emoticonsInfo){
            const emoRegex = new RegExp('\\b' + emo.name + '\\b')
            const newFragments: (string | EmoticonViewData)[] = []
            
            for(const oldFrag of fragments){
                if(typeof oldFrag == 'string'){
                    const textFragments = oldFrag.split(emoRegex)
                    
                    for(let i = 0; i < textFragments.length; i++){
                        const textFrag = textFragments[i]
                        if(textFrag.length > 0)
                            newFragments.push(textFrag)
                        
                        if(i != textFragments.length - 1)
                            newFragments.push({name: emo.name, fileName: emo.file})
                    }
                } else {
                    newFragments.push(oldFrag)
                }
            }
                
            fragments = newFragments
        }
        
        return fragments
    }
    
    private get timeString(): string{
        const displayTime = new Date(this.streamStartTime)
        displayTime.setTime(displayTime.getTime() + this.replayEvent.playerTimeMs)
        return displayTime.getHours().toString().padStart(2, '0') + ':' + displayTime.getMinutes().toString().padStart(2, '0')
    }
    
    private getSubscriptionBadgedName (months): string {
        const ret = this.jadiscoBadgesInfo.subscriber
            .filter(x => x.months <= months).slice(-1)[0].file
        return ret
    }
    
    private getRandomUserColor(nick: string): string{
        const rng = new Chance(nick)
        return this.USER_COLORS[rng.integer({min: 0, max: this.USER_COLORS.length-1})]
    }
    
}
</script>