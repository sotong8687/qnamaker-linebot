using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Web.Http;
using isRock.LineBot;

namespace StudyHostExampleLinebot.Controllers
{
    public class TestQnAController : isRock.LineBot.LineWebHookControllerBase
    {
        const string channelAccessToken = "bcopXB14gIVZyakKHwqM0cHAHnD6hKyciEkNEU7az3YzFPcjPthLNRYv0I1nnpBkOijRpMJLjnmcPU7Q+5OSYwl4couXwAULOWK/QMy1rBlWgrQOHzkm4sdwaiAamn4irdJ2mca+TiF7VXFXaRoLgQdB04t89/1O/w1cDnyilFU=";
        const string AdminUserId = "U95b08c393c33a6c81c5c80f786cf5d03";
        const string QnAKey = "5a84d21f-2b72-4f48-bf7b-541003f4f32d";
        const string QnAEndpoint = "https://sotong8687.azurewebsites.net/qnamaker/knowledgebases/57e423d7-62b8-4779-b882-d049f2d6cc83/generateAnswer";
        const string UnknowAnswer = "不好意思，您可以換個方式問嗎? 我不太明白您的意思...";

        [Route("api/TestQnA")]
        [HttpPost]
        public IHttpActionResult POST()
        {
            try
            {
                //設定ChannelAccessToken(或抓取Web.Config)
                this.ChannelAccessToken = channelAccessToken;
                //取得Line Event(範例，只取第一個)
                var LineEvent = this.ReceivedMessage.events.FirstOrDefault();
                //配合Line verify 
                if (LineEvent.replyToken == "00000000000000000000000000000000") return Ok();
                //回覆訊息
                if (LineEvent.type == "message")  
                {
                    var repmsg = "";
                    if (LineEvent.message.type == "text") //收到文字
                    {
                        //建立 MsQnAMaker Client
                        var helper = new isRock.MsQnAMaker.Client(
                            new Uri(QnAEndpoint), QnAKey);
                        var QnAResponse = helper.GetResponse(LineEvent.message.text.Trim());
                        var ret = (from c in QnAResponse.answers
                                   orderby c.score descending
                                   select c
                                ).Take(1);

                        var responseText = UnknowAnswer;
                        if (ret.FirstOrDefault().score > 0)
                            responseText = ret.FirstOrDefault().answer;
                        //回覆
                        this.ReplyMessage(LineEvent.replyToken, responseText);
                    }
                    if (LineEvent.message.type == "sticker") //收到貼圖
                        this.ReplyMessage(LineEvent.replyToken, 1, 2);

                    if (LineEvent.message.text.ToLower().Contains("template message"))
                    {
                        //define actions
                        var act1 = new isRock.LineBot.MessageAction()
                        { text = "test action1", label = "test action1" };
                        var act2 = new isRock.LineBot.MessageAction()
                        { text = "test action2", label = "test action2" };

                        var tmp = new isRock.LineBot.ButtonsTemplate()
                        {
                            text = "Button Template text",
                            title = "Button Template title",
                            thumbnailImageUrl = new Uri("https://i.imgur.com/wVpGCoP.png"),
                        };

                        tmp.actions.Add(act1);
                        tmp.actions.Add(act2);
                        //add TemplateMessage into responseMsgs
                        this.ReplyMessage(new isRock.LineBot.TemplateMessage(tmp));
                    }
                 }
                //response OK
                return Ok();
            }
            catch (Exception ex)
            {
                //如果發生錯誤，傳訊息給Admin
                this.PushMessage(AdminUserId, "發生錯誤:\n" + ex.Message);
                //response OK
                return Ok();
            }
        }

        private void ReplyMessage(TemplateMessage templateMessage)
        {
            throw new NotImplementedException();
        }
    }
}
