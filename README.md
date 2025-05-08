Add-Type -AssemblyName System.Windows.Forms
Add-Type -AssemblyName System.Drawing

$form = New-Object System.Windows.Forms.Form
$form.Text = "Teams Temizleyici"
$form.Size = New-Object System.Drawing.Size(400, 400)
$form.StartPosition = "CenterScreen"

# Durum kutusu
$statusBox = New-Object System.Windows.Forms.TextBox
$statusBox.Multiline = $true
$statusBox.ScrollBars = "Vertical"
$statusBox.Size = New-Object System.Drawing.Size(350, 180)
$statusBox.Location = New-Object System.Drawing.Point(20, 150)
$statusBox.ReadOnly = $true
$form.Controls.Add($statusBox)

# Butonlar
function Add-Button {
    param (
        [string]$text,
        [int]$x,
        [int]$y,
        [scriptblock]$action
    )
    $button = New-Object System.Windows.Forms.Button
    $button.Text = $text
    $button.Size = New-Object System.Drawing.Size(320, 30)
    $button.Location = New-Object System.Drawing.Point($x, $y)
    $button.Add_Click($action)
    $form.Controls.Add($button)
}

Add-Button "1. Teams Uygulamalarını Kaldır" 30 20 {
    $apps = @(
        "Microsoft Teams",
        "Teams Machine-Wide Installer",
        "Microsoft Teams (work or school)",
        "Microsoft Teams (Preview)"
    )

    foreach ($app in $apps) {
        $found = Get-WmiObject -Class Win32_Product | Where-Object { $_.Name -like "*$app*" }
        if ($found) {
            $statusBox.AppendText("Kaldırılıyor: $($found.Name)`r`n")
            $found.Uninstall() | Out-Null
        } else {
            $statusBox.AppendText("Bulunamadı: $app`r`n")
        }
    }
}

Add-Button "2. Geride Kalan Dosya ve Kayıtları Temizle" 30 60 {
    $paths = @(
        "$env:LOCALAPPDATA\Microsoft\Teams",
        "$env:APPDATA\Microsoft\Teams",
        "$env:ProgramData\Teams",
        "$env:ProgramFiles\Teams Installer",
        "$env:ProgramFiles (x86)\Teams Installer"
    )

    foreach ($path in $paths) {
        if (Test-Path $path) {
            Remove-Item -Recurse -Force -Path $path
            $statusBox.AppendText("Silindi: $path`r`n")
        }
    }

    $regPaths = @(
        "HKCU:\Software\Microsoft\Office\Teams",
        "HKCU:\Software\Microsoft\Teams",
        "HKLM:\Software\Microsoft\Office\Teams",
        "HKLM:\Software\WOW6432Node\Microsoft\Office\Teams"
    )

    foreach ($reg in $regPaths) {
        if (Test-Path $reg) {
            Remove-Item -Recurse -Force $reg
            $statusBox.AppendText("Silindi: $reg`r`n")
        }
    }
}

Add-Button "3. Yeni Teams'i Yükle" 30 100 {
    $installer = "$env:TEMP\TeamsSetup.exe"
    $url = "https://go.microsoft.com/fwlink/p/?LinkID=869428"
    Invoke-WebRequest -Uri $url -OutFile $installer -UseBasicParsing
    Start-Process -FilePath $installer -ArgumentList "/quiet" -Wait
    $statusBox.AppendText("Yükleme tamamlandı.`r`n")
}

Add-Button "Kalan Dosya Var mı Kontrol Et" 30 300 {
    $paths = @(
        "$env:LOCALAPPDATA\Microsoft\Teams",
        "$env:APPDATA\Microsoft\Teams",
        "$env:ProgramData\Teams"
    )

    foreach ($path in $paths) {
        if (Test-Path $path) {
            $statusBox.AppendText("KALAN BULUNDU: $path`r`n")
        } else {
            $statusBox.AppendText("Temiz: $path`r`n")
        }
    }
}

$form.Topmost = $true
[void]$form.ShowDialog()
