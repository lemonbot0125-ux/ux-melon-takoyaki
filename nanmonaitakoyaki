const CHATWORK_API_TOKEN = process.env.CWapitoken;
const { createClient } = require("@supabase/supabase-js");
const supabase = createClient(
  process.env.SUPABASE_URL,
  process.env.SUPABASE_ANON_KEY
);
const express = require("express");
const app = express();
const axios = require("axios");
const { CronJob } = require("cron");

new CronJob(
    "0 0 0 * * *",
    async () => {
        await dailymessagerank();
    },
    null,
    true,
    "Asia/Tokyo"
  );
  
  async function dailymessagerank() {
    try {
        rooms = await GetChatWorkRooms();
        for(const room of rooms) {
            roomId = room.room_id;
            roomname = room.name
            messagenum = room.message_num;
            await supabase
            .from("messagerank")
            .upsert([
                {
                roomId,
                roomname,
                messagenum,
                },
            ], { onConflict: ['roomId'] });
        };
    } catch(err) {
        console.error("dailymessagerank error", error.response ? error.response.data : error.message)
    }
  }

  async function GetChatWorkRooms() {
    try {
        const response = axios.get("https://api.chatwork.com/v2/rooms",
            {
                headers: {
                    Accept: "application/json",
                    "x-chatworktoken": CHATWORK_API_TOKEN,
                }
            }   
        )
        return response.data;
    } catch(err) {
        console.error("GetChatWorkRoom error", error.response ? error.response.data : error.message);
    }};

    app.post("/mention", (req, res) => {
        const {
            body,
            message_id: messageId, 
            room_id: roomId, 
            from_account_id: accountId
        } = req.body.webhook_event;
        command(body, messageId, roomId, accountId)
    })
    const commands = {
        ranking: messagerank,
    }
    async function command(body, messageId, roomId, accountId) {
        const message = body.replace(/\/.*?\/|\s+/g, "");
        const command = getCommand(body);
        if (command && commands[command]) {
            await commands[command](body, message, messageId, roomId, accountId);
        } else if (command) {
            return;
        }}
        function getCommand(body) {
            const pattern = /\/(.*?)\//;
            const match = body.match(pattern);
            return match ? match[1] : null;
        }

        async function messagerank() {
            try {
                rooms = await GetChatWorkRooms();
                const diffs = [];
                for(const room of rooms) {
                    roomId = room.room_id;
                    roomname = room.name
                    messagenum = room.message_num;
                    const { data, error } = await supabase
                    .from("messagerank")
                    .select("*")
                    .eq("roomId", roomId)
                    .single();

                    if (error) {
                        console.error(`Supabase fetch error for room ${roomId}:`, error.message);
                        continue;
                    }
                    const prevMessageNum = data?.messagenum || 0;
                    const diff = messagenum - prevMessageNum;
                    diffs.push({
                        roomId: rId,
                        roomname: rName,
                        current,
                        previous,
                        diff
                    });
                };
                diffs.sort((a, b) => b.diff - a.diff);

                const top8 = diffs.slice(0, 8);

                let messageText = "[info][title]ランキング[/title]";
                top8.forEach((room, index) => {
                messageText += `No.${index + 1}：${room.roomname}（+${room.diff}件）\n`;
                });
                messageText += "[/info]";
                
            } catch(err) {
                console.error("dailymessagerank error", error.response ? error.response.data : error.message)
            }
        }

        async function sendchatwork(ms, roomId) {
            try {
                await axios.post(
                `https://api.chatwork.com/v2/rooms/${roomId}/messages`,
                new URLSearchParams({ body: ms }),
                {
                    headers: {
                    "X-ChatWorkToken": CHATWORK_API_TOKEN,
                    "Content-Type": "application/x-www-form-urlencoded",
                    },
                }
                );
                console.log("メッセージ送信成功");
            } catch (error) {
                console.error(
                "Chatworkへのメッセージ送信エラー:",
                error.response?.data || error.message
                );
            }
            }
