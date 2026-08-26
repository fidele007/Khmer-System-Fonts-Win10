# Khmer font replacement

`replace_khmer_font.py` replaces the Khmer subsystem of a Windows system font
with the Khmer subsystem from a chosen replacement font. It preserves the
input font's internal names and can enlarge the Khmer glyphs with `--scale`.

The replacement font should be a TrueType/OpenType font with a `glyf` table
and Khmer shaping data. Google Sans is used in the examples below, but any
compatible font can be supplied.

## 1. Get the Windows system fonts

Run PowerShell as Administrator from the project directory. This copies the
original Windows fonts into a local `input-windows` directory without changing
the installed system fonts:

```powershell
$inputDir = Join-Path $PWD "input-windows"
New-Item -ItemType Directory -Force -Path $inputDir | Out-Null

$fontNames = @(
    "LeelUIsl.ttf",
    "LeelaUIb.ttf",
    "LeelawUI.ttf",
    "KhmerUI.ttf",
    "KhmerUIb.ttf"
)

foreach ($fontName in $fontNames) {
    Copy-Item -LiteralPath (Join-Path $env:WINDIR "Fonts\$fontName") `
        -Destination (Join-Path $inputDir $fontName) -Force
}
```

If a file is not found in `C:\Windows\Fonts`, locate the installed font in
Windows Settings or copy the matching original file from the project's
`backup` directory.

## 2. Prepare the replacement fonts

Extract the replacement font files and set the directory below to their
location. For Google Sans, the required files are:

```text
GoogleSans-Regular.ttf
GoogleSans-Italic.ttf
GoogleSans-Bold.ttf
```

For example:

```powershell
$replacementDir = "C:\Fonts\GoogleSans\static"
$outputDir = Join-Path $PWD "dist\Khmer_System_Fonts_GoogleSans"
New-Item -ItemType Directory -Force -Path $outputDir | Out-Null
```

Install the Python dependency once:

```powershell
python -m pip install fonttools
```

## 3. Convert each Windows font

The following commands use `1.25` to make Khmer approximately comparable to
the English cap height. Use `1.0` for the replacement font's original Khmer
size, or try another factor such as `1.15` or `1.30`.

```powershell
$scale = 1.25

python tools\replace_khmer_font.py --scale $scale `
    --pair "$inputDir\LeelUIsl.ttf" `
           "$replacementDir\GoogleSans-Italic.ttf" `
           "$outputDir\LeelUIsl.ttf"

python tools\replace_khmer_font.py --scale $scale `
    --pair "$inputDir\LeelaUIb.ttf" `
           "$replacementDir\GoogleSans-Bold.ttf" `
           "$outputDir\LeelaUIb.ttf"

python tools\replace_khmer_font.py --scale $scale `
    --pair "$inputDir\LeelawUI.ttf" `
           "$replacementDir\GoogleSans-Regular.ttf" `
           "$outputDir\LeelawUI.ttf"

python tools\replace_khmer_font.py --scale $scale `
    --pair "$inputDir\KhmerUI.ttf" `
           "$replacementDir\GoogleSans-Regular.ttf" `
           "$outputDir\KhmerUI.ttf"

python tools\replace_khmer_font.py --scale $scale `
    --pair "$inputDir\KhmerUIb.ttf" `
           "$replacementDir\GoogleSans-Bold.ttf" `
           "$outputDir\KhmerUIb.ttf"
```

The same script can process all five fonts in one invocation:

```powershell
python tools\replace_khmer_font.py --scale 1.25 `
    --pair "$inputDir\LeelUIsl.ttf" "$replacementDir\GoogleSans-Italic.ttf" "$outputDir\LeelUIsl.ttf" `
    --pair "$inputDir\LeelaUIb.ttf" "$replacementDir\GoogleSans-Bold.ttf" "$outputDir\LeelaUIb.ttf" `
    --pair "$inputDir\LeelawUI.ttf" "$replacementDir\GoogleSans-Regular.ttf" "$outputDir\LeelawUI.ttf" `
    --pair "$inputDir\KhmerUI.ttf" "$replacementDir\GoogleSans-Regular.ttf" "$outputDir\KhmerUI.ttf" `
    --pair "$inputDir\KhmerUIb.ttf" "$replacementDir\GoogleSans-Bold.ttf" "$outputDir\KhmerUIb.ttf"
```

## 4. Install the generated fonts

1. Keep the original files as a backup.
2. Select the five generated `.ttf` files in the output directory.
3. Right-click and choose **Install for all users**.
4. Allow Windows to replace the existing fonts.
5. Restart applications, or restart Windows if an application still uses the
   old font cache.

The generated files intentionally retain the original internal names, so do
not install the original and generated versions side by side unless Windows
asks you to replace the existing font.
