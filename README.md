import { useState, useRef, useCallback } from "react";
import { Link, useNavigate } from "react-router-dom";
import { base44 } from "@/api/base44Client";
import { motion, AnimatePresence } from "framer-motion";
import {
  LayoutDashboard,
  Car,
  Users,
  CalendarCheck,
  FileUp,
  TrendingUp,
  Receipt,
  History,
  AlertOctagon,
  Calendar,
  Menu,
  X,
  LogOut,
  Milestone,
  Package,
  ChevronDown,
  Trash2,
  ChevronLeft,
  UserPlus,
} from "lucide-react";

const navItems = [
  { label: "Dashboard", icon: LayoutDashboard, page: "Dashboard" },
  {
    label: "Rentals",
    icon: CalendarCheck,
    group: true,
    children: [
      { label: "Bookings", icon: CalendarCheck, page: "Bookings" },
      { label: "Booking History", icon: History, page: "BookingHistory" },
    ],
  },
  {
    label: "Inventory",
    icon: Package,
    group: true,
    children: [
      { label: "Fleet Management", icon: Car, page: "Fleet" },
      { label: "Driver Management", icon: Users, page: "Drivers" },
    ],
  },
  {
    label: "Financials",
    icon: TrendingUp,
    group: true,
    children: [
      { label: "Invoices", icon: Receipt, page: "Invoices" },
      { label: "Finances", icon: TrendingUp, page: "Finances" },
    ],
  },
  {
    label: "Compliance",
    icon: AlertOctagon,
    group: true,
    children: [
      { label: "Fines", icon: AlertOctagon, page: "Fines" },
      { label: "Expiries", icon: Calendar, page: "Expiries" },
      { label: "Tolls", icon: Milestone, page: "Tolls" },
      { label: "Toll Lookup", icon: Milestone, page: "TollLookup" },
    ],
  },
  {
    label: "Admin",
    icon: FileUp,
    group: true,
    children: [
      { label: "Import", icon: FileUp, page: "Import" },
    ],
  },
];

// Bottom nav tabs (primary 4)
const bottomNavItems = [
  { label: "Dashboard", icon: LayoutDashboard, page: "Dashboard" },
  { label: "Bookings", icon: CalendarCheck, page: "Bookings" },
  { label: "Fleet", icon: Car, page: "Fleet" },
  { label: "Drivers", icon: Users, page: "Drivers" },
];

// Page title map
const PAGE_TITLES = {
  Dashboard: "Dashboard",
  Bookings: "Bookings",
  Fleet: "Fleet",
  Drivers: "Drivers",
  Invoices: "Invoices",
  Finances: "Finances",
  Fines: "Fines",
  Expiries: "Expiries",
  BookingHistory: "History",
  TollLookup: "Toll Lookup",
  Tolls: "Tolls",
  Import: "Import",
  DriverSignup: "Driver Signup",
};

// Pages that are "top-level" (no back button)
const TOP_LEVEL_PAGES = ["Dashboard", "Bookings", "Fleet", "Drivers"];

const noSelectStyle = { userSelect: "none", WebkitUserSelect: "none" };

function NavItem({ item, currentPageName, onClick }) {
  const [open, setOpen] = useState(
    item.group && item.children?.some((c) => c.page === currentPageName)
  );
  if (item.group) {
    return (
      <div>
        <button
          onClick={() => setOpen(!open)}
          style={noSelectStyle}
          className="w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium text-sidebar-foreground hover:bg-sidebar-accent transition-colors min-h-[44px]"
        >
          <item.icon size={18} />
          <span className="flex-1 text-left">{item.label}</span>
          <ChevronDown size={14} className={`transition-transform ${open ? "rotate-180" : ""}`} />
        </button>
        {open && (
          <div className="ml-4 mt-0.5 space-y-0.5">
            {item.children.map((child) => (
              <Link
                key={child.page}
                to={`/${child.page}`}
                onClick={onClick}
                style={noSelectStyle}
                className={`flex items-center gap-3 px-3 py-2 rounded-lg text-sm font-medium transition-colors min-h-[44px] ${
                  currentPageName === child.page
                    ? "bg-primary text-primary-foreground"
                    : "text-sidebar-foreground hover:bg-sidebar-accent"
                }`}
              >
                <child.icon size={16} />
                {child.label}
              </Link>
            ))}
          </div>
        )}
      </div>
    );
  }
  return (
    <Link
      to={`/${item.page}`}
      onClick={onClick}
      style={noSelectStyle}
      className={`flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-colors min-h-[44px] ${
        currentPageName === item.page
          ? "bg-primary text-primary-foreground"
          : "text-sidebar-foreground hover:bg-sidebar-accent"
      }`}
    >
      <item.icon size={18} />
      {item.label}
    </Link>
  );
}

export default function Layout({ children, currentPageName }) {
  const [mobileOpen, setMobileOpen] = useState(false);
  const [showDeleteConfirm, setShowDeleteConfirm] = useState(false);
  const navigate = useNavigate();
  const tabHistories = useRef({});

  const navigateTab = useCallback((page) => {
    if (!tabHistories.current[page]) tabHistories.current[page] = [];
    tabHistories.current[page].push(window.location.pathname);
    navigate(`/${page}`);
  }, [navigate]);

  const pageTitle = PAGE_TITLES[currentPageName] || currentPageName;
  const isTopLevel = TOP_LEVEL_PAGES.includes(currentPageName);

  // DriverSignup is a standalone public page — no sidebar/nav
  if (currentPageName === "DriverSignup") {
    return <>{children}</>;
  }

  return (
    <div
      className="min-h-screen bg-background flex flex-col md:flex-row"
      style={{ overscrollBehaviorY: "none" }}
    >
      {/* ── Mobile Header ── */}
      <header
        className="md:hidden flex items-center justify-between px-4 bg-sidebar border-b border-sidebar-accent sticky top-0 z-40"
        style={{
          paddingTop: `calc(env(safe-area-inset-top) + 12px)`,
          paddingBottom: "12px",
          ...noSelectStyle,
        }}
      >
        {/* Left: back or logo */}
        <div className="flex items-center gap-2 min-w-[44px]">
          {!isTopLevel ? (
            <button
              onClick={() => window.history.back()}
              style={noSelectStyle}
              className="flex items-center gap-1 text-sidebar-foreground min-w-[44px] min-h-[44px] -ml-1"
            >
              <ChevronLeft size={22} />
              <span className="text-sm font-medium">Back</span>
            </button>
          ) : (
            <span className="text-sidebar-foreground font-bold text-base tracking-tight">
              Saga
            </span>
          )}
        </div>

        {/* Center: page title */}
        <span className="text-sidebar-foreground font-semibold text-base absolute left-1/2 -translate-x-1/2">
          {pageTitle}
        </span>

        {/* Right: menu toggle */}
        <button
          onClick={() => setMobileOpen(!mobileOpen)}
          style={noSelectStyle}
          className="text-sidebar-foreground min-w-[44px] min-h-[44px] flex items-center justify-end"
        >
          {mobileOpen ? <X size={22} /> : <Menu size={22} />}
        </button>
      </header>

      {/* ── Mobile Nav Overlay ── */}
      <AnimatePresence>
        {mobileOpen && (
          <motion.div
            key="overlay"
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
            className="md:hidden fixed inset-0 z-30 bg-black/60"
            onClick={() => setMobileOpen(false)}
          >
            <motion.nav
              initial={{ x: -280 }}
              animate={{ x: 0 }}
              exit={{ x: -280 }}
              transition={{ type: "tween", duration: 0.25 }}
              className="absolute top-0 left-0 h-full w-64 bg-sidebar flex flex-col pb-8 px-4 gap-1 overflow-y-auto"
              style={{ paddingTop: `calc(env(safe-area-inset-top) + 64px)` }}
              onClick={(e) => e.stopPropagation()}
            >
              <div className="mb-6 px-2">
                <img
                  src="https://qtrypzzcjebvfcihiynt.supabase.co/storage/v1/object/public/base44-prod/public/69a8dfbe36391dfbf5919088/8fb21d3dc_SAGA.png"
                  alt="Saga Automotive"
                  className="h-8 w-auto mb-1"
                  style={noSelectStyle}
                />
                <span className="text-sidebar-foreground font-bold text-xl tracking-tight" style={noSelectStyle}>
                  Saga Automotive
                </span>
                <p className="text-muted-foreground text-xs mt-1">Fleet & Payment Tracker</p>
              </div>

              {navItems.map((item) => (
                <NavItem
                  key={item.page || item.label}
                  item={item}
                  currentPageName={currentPageName}
                  onClick={() => setMobileOpen(false)}
                />
              ))}

              {/* Driver Signup */}
              <div className="mt-4 pt-4 border-t border-sidebar-accent">
                <Link
                  to="/DriverSignup"
                  onClick={() => setMobileOpen(false)}
                  style={noSelectStyle}
                  className={`flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-colors min-h-[44px] ${
                    currentPageName === "DriverSignup"
                      ? "bg-primary text-primary-foreground"
                      : "text-sidebar-foreground hover:bg-sidebar-accent"
                  }`}
                >
                  <UserPlus size={18} />
                  Driver Signup
                </Link>
              </div>

              {/* Logout + Delete Account */}
              <div className="mt-2 pt-2 border-t border-sidebar-accent space-y-1">
                <button
                  onClick={() => { base44.auth.logout(); setMobileOpen(false); }}
                  style={noSelectStyle}
                  className="w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium text-sidebar-foreground hover:bg-sidebar-accent transition-colors min-h-[44px]"
                >
                  <LogOut size={18} />
                  Logout
                </button>
                {!showDeleteConfirm ? (
                  <button
                    onClick={() => setShowDeleteConfirm(true)}
                    style={noSelectStyle}
                    className="w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium text-destructive hover:bg-destructive/10 transition-colors min-h-[44px]"
                  >
                    <Trash2 size={18} />
                    Delete Account
                  </button>
                ) : (
                  <div className="bg-destructive/10 rounded-lg p-3 space-y-2">
                    <p className="text-xs text-destructive font-medium">Are you sure? This cannot be undone.</p>
                    <div className="flex gap-2">
                      <button
                        onClick={() => setShowDeleteConfirm(false)}
                        style={noSelectStyle}
                        className="flex-1 text-xs py-2 rounded-md border border-border text-foreground min-h-[44px]"
                      >
                        Cancel
                      </button>
                      <button
                        onClick={() => alert("Please contact support@sagaautomotive.com.au to request account deletion.")}
                        style={noSelectStyle}
                        className="flex-1 text-xs py-2 rounded-md bg-destructive text-white min-h-[44px]"
                      >
                        Delete
                      </button>
                    </div>
                  </div>
                )}
              </div>
            </motion.nav>
          </motion.div>
        )}
      </AnimatePresence>

      {/* ── Desktop Sidebar ── */}
      <aside className="hidden md:flex flex-col w-60 bg-sidebar border-r border-sidebar-accent min-h-screen sticky top-0 h-screen">
        <div className="px-6 py-6 mb-2">
          <img
            src="https://qtrypzzcjebvfcihiynt.supabase.co/storage/v1/object/public/base44-prod/public/69a8dfbe36391dfbf5919088/8fb21d3dc_SAGA.png"
            alt="Saga Automotive"
            className="h-10 w-auto mb-2"
            style={noSelectStyle}
          />
          <span className="text-sidebar-foreground font-bold text-xl tracking-tight" style={noSelectStyle}>
            Saga Automotive
          </span>
          <p className="text-muted-foreground text-xs mt-1">Fleet & Payment Tracker</p>
        </div>
        <nav className="flex flex-col gap-1 px-3 flex-1 overflow-y-auto">
          {navItems.map((item) => (
            <NavItem key={item.page || item.label} item={item} currentPageName={currentPageName} />
          ))}
        </nav>
        <div className="px-3 mt-auto pt-4 space-y-1">
          <Link
            to="/DriverSignup"
            style={noSelectStyle}
            className={`flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium transition-colors min-h-[44px] ${
              currentPageName === "DriverSignup"
                ? "bg-primary text-primary-foreground"
                : "text-sidebar-foreground hover:bg-sidebar-accent"
            }`}
          >
            <UserPlus size={18} />
            Driver Signup
          </Link>
          <button
            onClick={() => base44.auth.logout()}
            style={noSelectStyle}
            className="w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium text-sidebar-foreground hover:bg-sidebar-accent transition-colors border-t border-sidebar-accent pt-4 min-h-[44px]"
          >
            <LogOut size={18} />
            Logout
          </button>
          {!showDeleteConfirm ? (
            <button
              onClick={() => setShowDeleteConfirm(true)}
              style={noSelectStyle}
              className="w-full flex items-center gap-3 px-3 py-2.5 rounded-lg text-sm font-medium text-destructive hover:bg-destructive/10 transition-colors min-h-[44px]"
            >
              <Trash2 size={18} />
              Delete Account
            </button>
          ) : (
            <div className="bg-destructive/10 rounded-lg p-3 space-y-2 mx-0">
              <p className="text-xs text-destructive font-medium">Are you sure?</p>
              <div className="flex gap-2">
                <button
                  onClick={() => setShowDeleteConfirm(false)}
                  style={noSelectStyle}
                  className="flex-1 text-xs py-2 rounded-md border border-border text-foreground min-h-[44px]"
                >
                  Cancel
                </button>
                <button
                  onClick={() => { setShowDeleteConfirm(false); alert("Please contact support@sagaautomotive.com.au to request account deletion."); }}
                  style={noSelectStyle}
                  className="flex-1 text-xs py-2 rounded-md bg-destructive text-white min-h-[44px]"
                >
                  Delete
                </button>
              </div>
            </div>
          )}
        </div>
        <div className="px-6 py-4 text-xs text-muted-foreground border-t border-sidebar-accent mt-4">
          Simple. Fast. Reliable.
        </div>
      </aside>

      {/* ── Main content ── */}
      <main className="flex-1 min-h-screen md:pb-0" style={{ paddingBottom: "calc(env(safe-area-inset-bottom) + 64px)" }}>
        {children}
      </main>

      {/* ── Mobile Bottom Navigation ── */}
      <nav
        className="md:hidden fixed bottom-0 left-0 right-0 z-40 bg-sidebar border-t border-sidebar-accent flex"
        style={{
          paddingBottom: "env(safe-area-inset-bottom)",
          ...noSelectStyle,
        }}
      >
        {bottomNavItems.map((item) => {
          const active = currentPageName === item.page;
          return (
            <button
              key={item.page}
              onClick={() => !active && navigateTab(item.page)}
              style={noSelectStyle}
              className={`flex-1 flex flex-col items-center justify-center gap-0.5 py-2 min-h-[56px] transition-colors ${
                active ? "text-primary" : "text-sidebar-foreground/60"
              }`}
            >
              <item.icon size={22} />
              <span className="text-[10px] font-medium">{item.label}</span>
            </button>
          );
        })}
      </nav>
    </div>
  );
}
