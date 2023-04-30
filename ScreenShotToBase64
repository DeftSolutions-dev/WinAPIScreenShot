using System;
using System.Collections;
using System.Runtime.InteropServices;
using UnityEngine;
using System.Text;
using System.Diagnostics;

public class ScreenShotToBase64
{
    private const int SRCCOPY = 0x00CC0020;

    [DllImport("user32.dll")]
    public static extern IntPtr GetDesktopWindow();

    [DllImport("user32.dll")]
    public static extern IntPtr GetWindowDC(IntPtr hWnd);

    [DllImport("user32.dll")]
    public static extern int ReleaseDC(IntPtr hWnd, IntPtr hDC);

    [DllImport("gdi32.dll")]
    public static extern IntPtr CreateCompatibleDC(IntPtr hDC);

    [DllImport("gdi32.dll")]
    public static extern IntPtr CreateCompatibleBitmap(IntPtr hDC, int nWidth, int nHeight);

    [DllImport("gdi32.dll")]
    public static extern IntPtr SelectObject(IntPtr hDC, IntPtr hObject);
    [DllImport("gdi32.dll")]
    public static extern int GetDIBits(IntPtr hdc, IntPtr hbm, uint start, uint cLines,
                                    IntPtr bits, ref BITMAPINFO bmi, uint usage);
    [DllImport("gdi32.dll")]
    public static extern bool BitBlt(IntPtr hDestDC, int xDest, int yDest, int
        wDest, int hDest, IntPtr hSrcDC, int xSrc, int ySrc, int
        RasterOp);

    [DllImport("gdi32.dll")]
    public static extern int DeleteDC(IntPtr hDC);

    [DllImport("gdi32.dll")]
    public static extern int DeleteObject(IntPtr hObject);

    [StructLayout(LayoutKind.Sequential, Pack = 2)]
    private struct BITMAPFILEHEADER
    {
        public static readonly short BM = 0x4d42;
        public short bfType;
        public int bfSize;
        public short bfReserved1;
        public short bfReserved2;
        public int bfOffBits;
    }
    [StructLayout(LayoutKind.Sequential)]
    private struct BITMAPINFOHEADER
    {
        public uint biSize;
        public int biWidth;
        public int biHeight;
        public ushort biPlanes;
        public ushort biBitCount;
        public uint biCompression;
        public uint biSizeImage;
        public int biXPelsPerMeter;
        public int biYPelsPerMeter;
        public uint biClrUsed;
        public uint biClrImportant;
    }
    [StructLayout(LayoutKind.Sequential)]
    private struct RGBQUAD
    {
        public byte rgbBlue;
        public byte rgbGreen;
        public byte rgbRed;
        public byte rgbReserved;
    }
    [StructLayout(LayoutKind.Sequential)]
    private struct BITMAPINFO
    {
        public BITMAPINFOHEADER bmiHeader;
        [MarshalAs(UnmanagedType.ByValArray, SizeConst = 256)]
        public RGBQUAD[] bmiColors;
    }
    public string CaptureScreen()
    {
        IntPtr desktopWindow = GetDesktopWindow();
        IntPtr desktopDc = GetWindowDC(desktopWindow);
        IntPtr memoryDc = CreateCompatibleDC(desktopDc);
        int screenWidth = Screen.width;
        int screenHeight = Screen.height;
        IntPtr bitmap = CreateCompatibleBitmap(desktopDc, screenWidth, screenHeight);
        if (bitmap != IntPtr.Zero)
        {
            IntPtr oldBitmap = SelectObject(memoryDc, bitmap);
            BitBlt(memoryDc, 0, 0, screenWidth, screenHeight, desktopDc, 0, 0, SRCCOPY);
            SelectObject(memoryDc, oldBitmap);
            BITMAPINFO bmi = new BITMAPINFO();
            bmi.bmiHeader.biSize = (uint)Marshal.SizeOf(typeof(BITMAPINFOHEADER));
            bmi.bmiHeader.biWidth = screenWidth;
            bmi.bmiHeader.biHeight = screenHeight;
            bmi.bmiHeader.biPlanes = 1;
            bmi.bmiHeader.biBitCount = 24;
            bmi.bmiHeader.biCompression = 0;
            bmi.bmiHeader.biSizeImage = (uint)(((screenWidth * 24 + 31) & ~31) / 8 * screenHeight);
            byte[] bitmapData = new byte[bmi.bmiHeader.biSizeImage];
            IntPtr bitmapPtr = IntPtr.Zero;
            try
            {
                bitmapPtr = Marshal.AllocHGlobal(bitmapData.Length);
                Marshal.StructureToPtr(bmi, bitmapPtr, false);
                if (BitBlt(memoryDc, 0, 0, screenWidth, screenHeight, desktopDc, 0, 0, SRCCOPY))
                {
                    if (GetDIBits(memoryDc, bitmap, 0, (uint)screenHeight, bitmapPtr, ref bmi, 0))
                    {
                        Marshal.Copy(bitmapPtr, bitmapData, 0, bitmapData.Length);
                        BITMAPFILEHEADER bmfh = new BITMAPFILEHEADER();
                        bmfh.bfType = BITMAPFILEHEADER.BM;
                        bmfh.bfSize = bitmapData.Length + Marshal.SizeOf(typeof(BITMAPFILEHEADER));
                        bmfh.bfReserved1 = 0;
                        bmfh.bfReserved2 = 0;
                        bmfh.bfOffBits = Marshal.SizeOf(typeof(BITMAPFILEHEADER)) + Marshal.SizeOf(typeof(BITMAPINFOHEADER));
                        byte[] fileHeader = new byte[Marshal.SizeOf(typeof(BITMAPFILEHEADER))];
                        IntPtr ptr = Marshal.AllocHGlobal(fileHeader.Length);
                        try
                        {
                            Marshal.StructureToPtr(bmfh, ptr, false);
                            Marshal.Copy(ptr, fileHeader, 0, fileHeader.Length);
                        }
                        finally
                        {
                            Marshal.FreeHGlobal(ptr);
                        }
                        byte[] data = new byte[fileHeader.Length + bitmapData.Length];
                        Array.Copy(fileHeader, data, fileHeader.Length);
                        Array.Copy(bitmapData, 0, data, fileHeader.Length, bitmapData.Length);
                        string base64String = Convert.ToBase64String(data);
                        return base64String;
                    }
                }
            }
            finally
            {
                if (bitmapPtr != IntPtr.Zero)
                    Marshal.FreeHGlobal(bitmapPtr);
                DeleteObject(bitmap);
                DeleteDC(memoryDc);
                ReleaseDC(desktopWindow, desktopDc);
            }
            return null;
        }
    }
}
