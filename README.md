### Hi there 👋

**charli02/charli02** is a ✨ _special_ ✨ repository because its `README.md` (this file) appears on your GitHub profile.

Here are some ideas to get you started:

- 🔭 I’m currently working on ...
- 🌱 I’m currently learning ...
- 👯 I’m looking to collaborate on ...
- 🤔 I’m looking for help with ...
- 💬 Ask me about ...
- 📫 How to reach me: ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

 public ActionResult CreateFileName()
        {
            string newFileName = Guid.NewGuid().ToString();
            return Json(new { data = newFileName });
        }
        
        -----------------------------------------------------
         function DownloadInvoice(InvoiceUID, BatchID) {
        ShowLoader();
        $.ajax({
            url: '@Url.Action("CreateFileName", "Operations")',
            type: 'POST',
            data: {},
            success: function (result) {
                $.ajax({
                    url: '@Url.Action("generateBillingInvoicePdf", "Billing")',
                    type: 'POST',
                    data: { FileName: result.data, BillingBatchUID: null, BillingBatchID: BatchID, InvoiceUID: InvoiceUID },
                    success: function (result1) {
                        HideLoader();
                        window.open("/Resources/Downloads/" + result.data + ".pdf", '_blank');
                    }
                });
            }
        });
    }
    
    ----------------------------------------------------------
    
    
